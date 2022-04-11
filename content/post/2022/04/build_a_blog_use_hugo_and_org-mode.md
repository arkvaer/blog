+++
title = "使用Org-mode 和 hugo 搭建博客"
date = 2022-04-10T00:55:00+08:00
lastmod = 2022-04-11T17:27:47+08:00
tags = ["org-mode"]
categories = ["org-mode"]
draft = false
toc = true
+++

## 开篇 {#开篇}

最近迷上了 Emacs 的 org-mode, 所以想以后通过 org-mode 来写博客, 经过一番研究, 就有了现在的这个博客(虽然一年多没有更新了),
下面就来说说这篇博客的搭建历程, 包括一些实用的技巧和踩过的坑.


## 环境搭建 {#环境搭建}

1.  hugo [Install Hugo](https://gohugo.io/getting-started/installing/)
2.  hugo 主题 [Hugo theme Stack](https://docs.stack.jimmycai.com/)
3.  ox-hugo
4.  pandoc (用于博客迁移, 将 markdown 转为 org 文件)


### 搭建步骤 {#搭建步骤}

1.  通过 hugo 初始化一个目录

    ```shell
         # blog 为你的博客目录名称
         hugo new site blog
    ```

2.  将刚生成的目录发布到 github 上, 至于如何将该目录发布到 github 上有很多种方法, 这里就不展开了

3.  挑选博客主题我这里选用的是: [Hugo theme Stack](https://docs.stack.jimmycai.com/)
    根据各自的主题文档配置修改配置文件, 这里因为情况不一样, 我就以 Stack 主题举例了
    1.  进入到博客根目录下的 `themes` 文件夹下, 运行以下命令, 将主题仓库作为博客仓库的一个子模块

        ```bash
                git submodule add https://github.com/CaiJimmy/hugo-theme-stack/ themes/hugo-theme-stack
        ```

    2.  进入到主题目录下的 exampleSite 文件夹下 `/blog/themes/hugo-theme-stack/exampleSite/` 将其中的 `content` 文件夹和 `config.yaml` 文件覆盖到博客根目录下.

    3.  根据自己的喜好修改配置文件

最终的目录结构如下:

```text
 .
├──  archetypes
├──  config.yaml
├──  content
├──  data
├──  layouts
├──  LICENSE
├──  org
├──  public
├──  README.org
├──  resources
├──  static
└──  themes
   └──  hugo-theme-stack
```


## 工具配置 {#工具配置}

因为我使用的是 `Doom Emacs` 这里也就以此为例

1.  安装 ox-hugo
    在 `~/.doom.d/packages.el` 中添加下面代码, 以引入 ox-hugo 依赖

    ```elisp
        ;; 添加 ox-hugo
        (package! ox-hugo)
    ```

    在 `~/.doom.d/config.el` 文件中加入以下代码, 配置 ox-hugo 加载的时间

    ```elisp
        ;; hugo org 转 markdown
        (use-package! ox-hugo
          :after ox
            :config
        (setq
       ;; 设置在文件导出到content下的哪个目录，我的这个是 content/post
       org-hugo-section "post"
       ;; 设置文章的最后修改时间为导出时间
       org-hugo-auto-set-lastmod	t
       )
      ;; 设置在org文件保存的时候自动导出到org-hugo-section下
        (org-hugo-auto-export-mode)
          )
    ```
2.  添加代码片段(snippet), 方便以后创建文章使用在  `~/.doom.d/init.el` 中将 `:editor snippets` 注释去掉, 启用模块. 使用 `M-x doom/reload` 下载并重新加载模块然后使用 `M-x yas-new-snippet` 创建一个 snippet
    将下面代码片段复制到 snippet 编辑区

    ```nil
        # -*- mode: snippet -*-
        # name: hugo
        # uuid: edb130fa-6a6c-452b-8d2d-52b0400241fe
        # key: hugo
        # condition: t
        # --
        #+OPTIONS: author:nil ^:{}
        #+HUGO_BASE_DIR: ../../../blog
        #+HUGO_SECTION: post/`(format-time-string "%Y/%m")`
        #+HUGO_CUSTOM_FRONT_MATTER: :toc true
        #+HUGO_AUTO_SET_LASTMOD: t
        #+HUGO_DRAFT: false
        #+DATE: `(format-time-string "[%Y-%m-%d %a %H:%M]")`
        #+TITLE: $1
        #+HUGO_TAGS: $2
        #+HUGO_CATEGORIES: $3

        $0
    ```

注意:

1.  `#+HUGO_BASE_DIR` 的值以自己的为准, 也可以使用绝对路径, 指定的目录就是你的博客根目录
2.  `#+DATE` 的时间格式最好不要改动, 中间的 `%a` 代表的是周几, 而且当光标处于日期上时, 通过快捷键 `Shift+左/右方向键` 可以以天为单位前后调整日期.

编辑好 snippet 后通过快捷键 `C-c C-c` 来创建在 `Choose or enter a table (yas guesses nothing):` 中输入 `org-mode`
然后在 `File to save snippet in: ~/.doom.d/snippets/org-mode/` 后面输入 `__hugo.org` 保存结束即可建立成功后在 org 文件中 通过输入 `hugo` 关键字来快速插入代码片段效果如下:

![](/ox-hugo/01.png)
![](/ox-hugo/02.png)
也可以通过 `M-x yas-insert-snippet` 或通过快捷键 `C-c & C-s` 然后通过关键字 hugo 进行选用.


## 文件转换和生成 {#文件转换和生成}

使用 `C-c C-e H A` 将文件转为 markdown 文件


### 自动导出, 实时预览 {#自动导出-实时预览}

在 blog 根目录创建 `.dir-locals.el`
文件内容如下:

```elisp
;; .dir-locals.el
(("org/"
  . ((org-mode . ((eval . (org-hugo-auto-export-mode)))))))
```

其中 `org` 是你存放 org 文件的目录.


## 生成静态文件和部署 {#生成静态文件和部署}

1.  生成:
    在博客根目录使用命令 `hugo -D` 生成静态文件, 提交推送到 github 仓库
2.  部署到 [vercel](https://vercel.com):
    1.  选择要部署的仓库
        ![](/ox-hugo/03.png)
    2.  选择项目类型, 这里选择 Other 就行
        ![](/ox-hugo/04.png)
    3.  选择发布根目录
        ![](/ox-hugo/05.png)
    4.  自定义域名
        ![](/ox-hugo/06.png)
    5.  修改 vervel 分配的域名
        ![](/ox-hugo/07.png)
    6.  添加自定义域名
        ![](/ox-hugo/08.png)
    7.  选择重定向方式三种方式分别是:

        -   xxx.com --&gt; www.xxx.com
        -   www.xxx.com --&gt; xxx.com
        -   只添加 xxx.com 不进行重定向

        {{< figure src="/ox-hugo/09.png" >}}
