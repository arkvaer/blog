+++
title = "使用 Kind + Podman 搭建本地 Kubernetes 多节点集群"
date = 2017-12-25T19:25:00+08:00
lastmod = 2022-04-13T09:04:59+08:00
tags = ["云原生"]
categories = ["云原生"]
draft = false
toc = true
+++

想系统学习 Kubernetes，单节点环境只能解决最基础的 Pod、Deployment 和 Service 实验。当开始涉及 **Node、Service、负载均衡、滚动更新、扩缩容以及容器网络**时，一个多节点 Kubernetes 集群会更有价值。

生产环境通常使用 kubeadm、RKE2、K3s 或云厂商 Kubernetes 服务，但对于本地学习和实验，直接部署完整 Kubernetes 集群成本较高。

本文采用：
- Fedora Linux
- Podman
- Kind
- kubectl

搭建一个包含 **1 个 Control Plane + 2 个 Worker** 的本地 Kubernetes 集群，并在此基础上完成镜像管理、Service 暴露、负载均衡、Deployment 扩缩容和滚动更新。

---

## 一、为什么选择 Kind + Podman

[Kind](https://kind.sigs.k8s.io/)（Kubernetes IN Docker）本质上是一个用于创建 Kubernetes 测试集群的工具。

它与传统 Kubernetes 部署方式最大的区别是：

> **Kind 不需要为每个 Kubernetes Node 创建完整虚拟机，而是直接将 Kubernetes Node 运行在容器中。**

因此，一个三节点 Kubernetes 集群可以抽象成：

```text
┌──────────────────────────────────────────┐
│                Fedora Host               │
│                                          │
│                 Podman                   │
│                    │                     │
│        ┌───────────┼───────────┐         │
│        │           │           │         │
│   control-plane  worker      worker      │
│        │           │           │         │
│     Kubernetes  Kubernetes  Kubernetes   │
│        Node        Node        Node      │
└──────────────────────────────────────────┘
```

这种架构非常适合学习 Kubernetes，因为 Kubernetes 的 Node、Pod、Service、Deployment 等核心概念依然存在，只是 Node 的物理载体从虚拟机变成了容器。

而 Podman 本身支持 Rootless Container，因此可以进一步降低本地实验环境对 root 权限的依赖。

---

# 二、安装基础环境

首先安装 Podman、网络相关组件、kubectl 和 Docker Compose：

```bash
sudo dnf install podman passt kubectl docker-compose podman-docker
```

然后安装 Kind：

```bash
go install sigs.k8s.io/kind@v0.33.0
```

确认安装：

```bash
podman --version
kubectl version --client
kind version
```

这里有一个容易忽略的问题：

**Kind 的容器运行时和 Kubernetes 本身是两个不同层次的概念。**

可以简单理解为：

```text
kubectl
   │
   ▼
Kubernetes API Server
   │
   ▼
Kubernetes Cluster
   │
   ▼
Kind Node
   │
   ▼
Podman Container Runtime
```

kubectl 操作的是 Kubernetes API，而 Podman 负责承载 Kind Node 容器。

---

# 三、创建多节点 Kubernetes 集群

## 3.1 创建 Kind 配置文件

创建 `kind.yaml`：

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4

nodes:
  - role: control-plane
    extraPortMappings:
      - containerPort: 80
        hostPort: 8080
        protocol: TCP

      - containerPort: 443
        hostPort: 8443
        protocol: TCP

  - role: worker

  - role: worker
```

这个配置创建：

```text
Control Plane × 1
Worker × 2
```

同时把 Control Plane 容器的：

```text
80  →  宿主机 8080
443 →  宿主机 8443
```

进行端口映射。

需要注意：

`extraPortMappings` 是 **Kind Node 容器与宿主机之间的端口映射**，并不等同于 Kubernetes Service 的 `NodePort`。

这是 Kind 使用过程中非常容易混淆的两个概念。

---

## 3.2 创建集群

执行：

```bash
kind create cluster \
  --name dev \
  --config kind.yaml
```

如果使用 Podman 作为 Kind Provider，可以看到类似：

```text
using podman due to KIND_EXPERIMENTAL_PROVIDER
enabling experimental podman provider
```

创建完成后查看集群：

```bash
kind get clusters
```

查看 Kubernetes Node：

```bash
kubectl get nodes -o wide
```

正常情况下可以看到：

```text
NAME                  STATUS   ROLES           AGE
dev-control-plane     Ready    control-plane   ...
dev-worker            Ready    <none>          ...
dev-worker2           Ready    <none>          ...
```

至此，一个三节点 Kubernetes 集群已经运行起来。

---

# 四、Kind 中的镜像管理

在本地使用 Kind 时，一个非常常见的问题是：

> 宿主机明明已经有这个镜像，为什么 Kubernetes 创建 Pod 时仍然需要拉取镜像？

原因在于：

**宿主机的容器镜像存储与 Kind Node 内部的容器镜像存储并不是同一个环境。**

架构可以理解为：

```text
Fedora Host
   │
   └── Podman Image Store
          │
          │  podman pull
          ▼
     Kubernetes Image

Kind Cluster
   │
   ├── dev-control-plane
   ├── dev-worker
   └── dev-worker2
          │
          └── Node 内部的 Container Runtime
```

因此：

```bash
podman pull xxx
```

只是将镜像拉到了宿主机的镜像存储中。

Kubernetes Pod 真正启动时，需要 Node 内部能够访问这个镜像。

---

# 五、使用 kind load 加载本地镜像

对于无法直接访问 Docker Hub、GCR 等公网 Registry 的环境，一个非常实用的方法是：

> **先在宿主机拉取镜像，再使用 `kind load docker-image` 将镜像加载到 Kind 集群。**

例如：

```bash
podman pull gcr.io/google-samples/kubernetes-bootcamp:v1
```

然后：

```bash
kind load docker-image \
  gcr.io/google-samples/kubernetes-bootcamp:v1 \
  --name dev
```

最后创建 Deployment：

```bash
kubectl create deployment kubernetes-bootcamp \
  --image=gcr.io/google-samples/kubernetes-bootcamp:v1
```

这种方式特别适合以下场景：

- 国内网络环境无法直接访问海外 Registry
- 本地开发镜像
- Kubernetes 实验环境
- CI/CD 测试
- 不希望搭建私有 Registry 的临时实验

---

## 5.1 为什么不要使用 `latest`

这里有一个非常重要的实践原则：

> **Kind 加载本地镜像时，尽量使用明确的版本 Tag，不要使用 `latest`。**

例如：

```text
推荐：

kubernetes-bootcamp:v1

不推荐：

kubernetes-bootcamp:latest
```

原因在于 `latest` 并不是一个稳定的版本标识。

如果 Kubernetes 认为本地镜像不可用，或者 Pod 的 `imagePullPolicy` 导致重新拉取镜像，就可能再次访问远程 Registry。

因此，在本地实验环境中，最好始终使用明确版本：

```yaml
image: kubernetes-bootcamp:v1
```

而不是：

```yaml
image: kubernetes-bootcamp:latest
```

**版本化镜像 Tag 是可重复实验的重要基础。**

---

# 六、解决 Kind 节点无法拉取镜像的问题

如果希望 Kubernetes Node 自己通过网络拉取镜像，就需要解决 Kind Node 的网络访问问题。

例如：

```bash
kind create cluster \
  --name dev \
  --config kind.yaml
```

进入 Control Plane：

```bash
podman exec -it dev-control-plane bash
```

然后为 containerd 配置代理：

```bash
mkdir -p /etc/systemd/system/containerd.service.d
```

创建：

```bash
cat >/etc/systemd/system/containerd.service.d/proxy.conf <<'EOF'
[Service]
Environment="HTTP_PROXY=http://host.containers.internal:10808"
Environment="HTTPS_PROXY=http://host.containers.internal:10808"
Environment="NO_PROXY=localhost,127.0.0.1,10.88.0.0/16,10.96.0.0/12,10.244.0.0/16,.svc,.svc.cluster.local"
EOF
```

这里有几个关键点。

### `host.containers.internal`

这是容器访问宿主机的一种特殊主机名。

在本例中：

```text
Kind Node
   │
   │ HTTP_PROXY
   ▼
host.containers.internal:10808
   │
   ▼
Fedora Host
   │
   ▼
代理服务
```

这样 Kind Node 内部的 containerd 就可以通过宿主机代理访问外部 Registry。

---

## 6.1 NO_PROXY 为什么重要

代理配置不能简单地写成：

```text
所有流量 → Proxy
```

Kubernetes 集群内部存在大量不应该经过外部代理的地址，例如：

```text
localhost
127.0.0.1
Service CIDR
Pod CIDR
.svc
.svc.cluster.local
```

因此需要配置：

```text
NO_PROXY
```

例如：

```text
10.96.0.0/12
```

通常对应 Kubernetes Service 网络。

而：

```text
10.244.0.0/16
```

则可以对应 Pod 网络。

实际 CIDR 应以当前 Kind/Kubernetes 集群配置为准。

---

# 七、系统重启后如何恢复 Kind 集群

Kind Node 本质上是 Podman 容器。

因此系统重启之后，如果容器没有自动启动，可以手动启动：

```bash
podman start dev-control-plane
podman start dev-worker
podman start dev-worker2
```

然后检查：

```bash
kubectl get nodes
```

如果 Node 状态恢复为：

```text
Ready
```

说明 Kubernetes 集群已经恢复。

这里需要注意：

**不要把 `kind create cluster` 当成集群启动命令。**

`kind create cluster` 用于创建集群，而已经存在的 Kind Node 可以直接通过容器运行时重新启动。

---

# 八、kubectl 默认 Namespace
由于每次重启后 Kubernetes 都会使用default 作为默认Namespace, 为避免后续频繁切换Namespace, 可以修改默认的Namespeace

Kubernetes 中的资源默认属于当前 Namespace。

查看所有 Namespace：

```bash
kubectl get namespaces
```

查看所有 Namespace 中的 Pod：

```bash
kubectl get pods -A
```

修改当前 kubectl Context 的默认 Namespace：

```bash
kubectl config set-context \
  --current \
  --namespace=kube-system
```

之后：

```bash
kubectl get pods
```

默认查询的就是：

```text
kube-system
```

而不是：

```text
default
```

可以通过：

```bash
kubectl config view --minify
```

检查当前 Context 配置。

需要明确：

> **Namespace 是 Kubernetes API 资源的逻辑隔离边界，不是网络隔离边界。**

因此修改默认 Namespace 只是改变 kubectl 的默认操作范围，并不会改变 Pod 的网络拓扑。

---

# 九、使用 Service 暴露 Deployment

创建 Deployment 后，可以通过 Service 为 Pod 提供稳定的访问入口。

例如：

```bash
kubectl expose deployment kubernetes-bootcamp \
  --type=NodePort \
  --port=8080
```

查看 Service：

```bash
kubectl get service
```

此时 Kubernetes 会创建一个 Service。

Service 的核心价值是：

> **为一组动态变化的 Pod 提供稳定的虚拟访问入口。**

Pod IP 会随着 Pod 创建、删除、重启而变化。

例如：

```text
Pod A
10.244.0.5

Pod B
10.244.1.8

Pod C
10.244.2.4
```

客户端不应该直接依赖这些 Pod IP。

而应该访问：

```text
Service
    │
    ├── Pod A
    ├── Pod B
    └── Pod C
```

Service 通过 Selector 选择后端 Pod，并维护对应的 EndpointSlice。

---

# 十、使用 port-forward 访问 Service

开发和调试过程中，可以直接使用：

```bash
kubectl port-forward \
  service/kubernetes-bootcamp \
  8080:8080
```

默认情况下，`kubectl port-forward` 通常只监听本机回环地址。

如果希望其他机器也能够访问，可以指定：

```bash
kubectl port-forward \
  service/kubernetes-bootcamp \
  --address 0.0.0.0 \
  18080:8080
```

此时访问：

```text
宿主机IP:18080
```

即可转发到 Kubernetes Service。

需要强调：

> `kubectl port-forward` 更适合开发、调试和临时访问，不应该作为生产环境的服务暴露方案。

它建立的是：

```text
客户端
  │
  ▼
kubectl
  │
  ▼
Kubernetes API
  │
  ▼
Service / Pod
```

而不是典型的生产流量入口架构。

---

# 十一、为什么宿主机测试 Service 负载均衡可能产生误导

这是本实验中比较容易踩坑的一点。

在：

```text
Rootless Podman + Kind
```

环境下，宿主机、Kind Node 和 Kubernetes Pod 实际处于不同的网络层。

可以抽象为：

```text
                 Fedora Host
                     │
              Podman Network
                     │
        ┌────────────┼────────────┐
        │            │            │
   Control Plane   Worker       Worker
        │            │            │
        └──────── Kubernetes ─────┘
                     │
                   Service
                     │
             ┌───────┼───────┐
             ▼       ▼       ▼
            Pod     Pod     Pod
```

因此：

**宿主机访问 Service 所经过的网络路径，与 Kubernetes 集群内部 Pod 访问 Service 并不完全相同。**

如果实验目标是验证 Kubernetes Service 的后端 Pod 分发，最直接的方法是：

> **从集群内部发起请求。**

这样可以避免宿主机到 Kind Node 的额外网络路径对实验结果造成干扰。

---

# 十二、从集群内部验证 Service 负载均衡

首先准备一个 curl 镜像：

```bash
podman pull curlimages/curl:8.21.0
```

加载到 Kind：

```bash
kind load docker-image \
  docker.io/curlimages/curl:8.21.0 \
  --name dev
```

创建测试 Pod：

```bash
kubectl run test-curl \
  --image=curlimages/curl:8.21.0 \
  -it --rm -- sh
```

进入 Pod 后执行：

```bash
curl http://kubernetes-bootcamp:8080
```

这里直接使用：

```text
kubernetes-bootcamp
```

而不是 ClusterIP。

原因是 Kubernetes 默认提供 Service DNS。

因此：

```text
kubernetes-bootcamp
        │
        ▼
Kubernetes DNS
        │
        ▼
Service ClusterIP
        │
        ▼
EndpointSlice
        │
        ▼
Backend Pods
```

多次执行：

```bash
curl http://kubernetes-bootcamp:8080
```

即可观察请求是否被分发到不同的 Pod。

---

# 十三、Service 到底是如何实现负载均衡的？

从 Kubernetes 的抽象模型来看：

```text
Deployment
    │
    ▼
ReplicaSet
    │
    ▼
Pods
    │
    ▼
Service Selector
    │
    ▼
EndpointSlice
    │
    ▼
Service
```

假设 Deployment 有三个 Pod：

```text
kubernetes-bootcamp-xxx-1
kubernetes-bootcamp-xxx-2
kubernetes-bootcamp-xxx-3
```

Service 通过 Selector 找到这些 Pod，并将其作为后端 Endpoint。

客户端访问：

```text
kubernetes-bootcamp:8080
```

首先通过 Kubernetes DNS 解析 Service。

然后由 Kubernetes 节点侧的数据平面组件将 Service 流量转发到对应的后端 Pod。

因此 Service 最重要的价值并不是简单的“负载均衡器”，而是提供：

1. 稳定的虚拟 IP
2. 稳定的 DNS 名称
3. Pod 后端动态发现
4. 后端 Endpoint 生命周期管理
5. 流量转发

这也是为什么客户端不应该直接依赖 Pod IP。

---

# 十四、Deployment 扩容与缩容

当业务流量增加时，可以增加 Deployment 的 Pod 副本数。

例如扩容到 4 个副本：

```bash
kubectl scale deployments/kubernetes-bootcamp \
  --replicas=4
```

查看：

```bash
kubectl get pods
```

缩容到 2 个副本：

```bash
kubectl scale deployments/kubernetes-bootcamp \
  --replicas=2
```

Deployment 并不是直接管理 Pod，而是通过 ReplicaSet 管理 Pod。

整体关系：

```text
Deployment
     │
     ▼
 ReplicaSet
     │
 ┌───┼────┐
 ▼   ▼    ▼
Pod Pod  Pod
```

执行：

```bash
kubectl scale
```

本质上是在修改 Deployment 的期望副本数。

例如：

```text
replicas = 4
```

表示 Kubernetes Controller 希望集群最终保持：

```text
4 个符合条件的 Pod
```

如果当前只有 2 个：

```text
Current = 2
Desired = 4
```

Controller 就会创建新的 Pod。

反之：

```text
Current = 4
Desired = 2
```

Controller 就会删除多余 Pod。

这体现了 Kubernetes 最核心的设计思想之一：

> **声明式 API + Controller Reconciliation。**

用户声明“我希望有多少个副本”，Controller 负责不断将实际状态向期望状态收敛。

---

# 十五、Deployment 滚动更新

Kubernetes 不仅可以扩缩容，还可以在不中断业务的情况下更新应用版本。

例如当前使用：

```text
kubernetes-bootcamp:v1
```

准备更新到：

```text
kubernetes-bootcamp:v2
```

首先准备镜像：

```bash
podman pull docker.io/jocatalin/kubernetes-bootcamp:v2
```

然后加载到 Kind：

```bash
kind load docker-image \
  docker.io/jocatalin/kubernetes-bootcamp:v2 \
  --name dev
```

修改 Deployment：

```bash
kubectl set image deployments/kubernetes-bootcamp \
  kubernetes-bootcamp=docker.io/jocatalin/kubernetes-bootcamp:v2
```

查看 Pod：

```bash
kubectl get pods
```

也可以进一步：

```bash
kubectl describe pods
```

观察 Pod 的镜像版本变化。

---

# 十六、Rolling Update 的工作机制

Deployment 默认采用 RollingUpdate 策略。

它不是简单地：

```text
删除所有旧 Pod
        ↓
创建所有新 Pod
```

而是：

```text
旧版本 Pod
    │
    ├── 创建新版本 Pod
    │
    ├── 等待新 Pod Ready
    │
    ├── 删除部分旧 Pod
    │
    ├── 创建更多新 Pod
    │
    └── 最终全部切换到新版本
```

最终：

```text
v1 Pod × 0
v2 Pod × N
```

因此，在合理配置：

- `maxSurge`
- `maxUnavailable`
- Readiness Probe

的情况下，可以实现服务的平滑版本切换。

这也是 Kubernetes Deployment 相比直接运行 Pod 的重要优势。

---

# 十七、如何回滚 Deployment

如果新版本存在问题，可以将镜像重新切换回旧版本。

例如：

```bash
kubectl set image deployments/kubernetes-bootcamp \
  kubernetes-bootcamp=gcr.io/google-samples/kubernetes-bootcamp:v10
```

然后：

```bash
kubectl describe pods
```

检查 Pod 使用的镜像版本。

在实际生产环境中，更推荐结合 Deployment Revision 使用：

```bash
kubectl rollout history deployment/kubernetes-bootcamp
```

查看历史版本：

```bash
kubectl rollout history deployment/kubernetes-bootcamp
```

出现问题时：

```bash
kubectl rollout undo deployment/kubernetes-bootcamp
```

即可回滚到上一版本。

因此，一个典型的 Kubernetes 应用发布流程可以抽象为：

```text
Build Image
     │
     ▼
Push Registry
     │
     ▼
Update Deployment
     │
     ▼
Rolling Update
     │
     ├── Pod Ready
     │
     ├── Service 接收流量
     │
     └── Old Pod 逐步退出
              │
              ▼
          Release Done
```

如果新版本异常：

```text
Deployment
     │
     ▼
rollout undo
     │
     ▼
Previous Revision
```

---

# 十八、完整实验链路

到这里，我们实际上已经完成了一套比较完整的 Kubernetes 基础实验。

整体流程如下：

```text
                 Kind + Podman
                       │
                       ▼
              Kubernetes Cluster
                       │
             ┌─────────┴─────────┐
             │                   │
        Control Plane          Workers
             │                   │
             └─────────┬─────────┘
                       │
                   Deployment
                       │
                       ▼
                   ReplicaSet
                       │
             ┌─────────┼─────────┐
             ▼         ▼         ▼
            Pod       Pod       Pod
             │         │         │
             └─────────┼─────────┘
                       │
                    Service
                       │
                       ▼
                  EndpointSlice
                       │
                       ▼
                 Service Traffic
                       │
                       ▼
                  Backend Pods
```

然后通过：

```bash
kubectl scale
```

改变副本数量。

通过：

```bash
kubectl set image
```

修改应用版本。

通过：

```bash
kubectl rollout
```

管理 Deployment 发布历史。

通过：

```bash
kubectl port-forward
```

进行本地调试。

通过：

```bash
kubectl expose
```

创建 Service。

---

# 十九、几个值得记住的 Kubernetes 实践原则

## 1. Pod 不是稳定的服务入口

Pod IP 是临时的。

不要让业务客户端直接依赖 Pod IP。

应该使用：

```text
Service DNS
```

作为稳定访问入口。

---

## 2. Deployment 不等于 Pod

Deployment 负责声明应用的期望状态。

实际管理链路是：

```text
Deployment
    ↓
ReplicaSet
    ↓
Pod
```

理解这个关系，是理解 Kubernetes Controller 模型的基础。

---

## 3. Service 不负责创建 Pod

Service 和 Deployment 是两个完全不同的 API Object。

```text
Deployment
    └── 管理 Pod

Service
    └── 管理 Pod 的访问入口
```

Service 通过 Selector 发现后端 Pod。

---

## 4. `port-forward` 主要用于调试

```bash
kubectl port-forward
```

非常适合：

- 本地开发
- Debug
- 临时访问
- 快速验证服务

但它不是生产环境的标准流量入口。

生产环境通常需要考虑：

```text
Ingress
Gateway
LoadBalancer
NodePort
Service Mesh
```

等方案。

---

## 5. 本地 Kind 集群和生产 Kubernetes 不完全等价

Kind 非常适合：

- Kubernetes 学习
- API 实验
- Controller 开发
- CI 测试
- Helm 实验
- Service Mesh 学习

但它毕竟运行在本地容器环境中。

尤其是：

```text
Rootless Podman
+
Kind
+
Kubernetes Networking
```

会引入额外的网络层。

因此，在进行网络实验时，需要明确区分：

```text
宿主机网络
   ↓
Podman 网络
   ↓
Kind Node 网络
   ↓
Kubernetes Pod 网络
```

很多“为什么我的 Kubernetes 行为和教程不一样”的问题，本质上并不是 Kubernetes 本身的问题，而是**运行环境的网络拓扑不同**。

---

# 二十、总结

使用 Kind + Podman，可以在一台普通 Linux 主机上快速构建一个多节点 Kubernetes 实验环境。

本文从一个最简单的 `kubernetes-bootcamp` 应用出发，逐步完成：

```text
安装 Podman / kubectl / Kind
            ↓
创建三节点 Kubernetes 集群
            ↓
解决镜像拉取问题
            ↓
理解 Kind 镜像管理
            ↓
配置容器代理
            ↓
管理 Namespace
            ↓
创建 Service
            ↓
port-forward 暴露服务
            ↓
集群内部验证 Service
            ↓
验证 Pod 流量分发
            ↓
Deployment 扩缩容
            ↓
Deployment Rolling Update
            ↓
版本回滚
```

如果是为了系统学习 Kubernetes，这套实验的重点并不是记住几个 `kubectl` 命令，而是建立下面这套模型：

```text
             Kubernetes API
                   │
                   ▼
              Deployment
                   │
                   ▼
               ReplicaSet
                   │
                   ▼
                  Pod
                   │
             ┌─────┴─────┐
             │           │
        Pod IP       Pod IP / Pod IP
             │           │
             └─────┬─────┘
                   │
                Service
                   │
                   ▼
             EndpointSlice
                   │
                   ▼
             Service Traffic
```

一旦理解了 **Deployment → ReplicaSet → Pod** 和 **Service → EndpointSlice → Pod** 这两条核心链路，再继续学习 Kubernetes 的 Ingress、Gateway、HPA、StatefulSet、DaemonSet，以及 Istio 等 Service Mesh 技术，就会容易很多。

对于本地 Kubernetes 学习环境而言，**Kind + Podman 是一个成本低、可重复、足够接近真实 Kubernetes 对象模型的实验方案**。
