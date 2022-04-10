+++
title = "IDEA性能调优"
date = 2018-03-30T22:15:00+08:00
lastmod = 2022-04-10T01:46:58+08:00
tags = ["IDE"]
categories = ["IDE"]
draft = false
toc = true
+++

优化参数如下:

```text
-server
-Xms1024m
-Xmx4096m
-XX:NewRatio=3
-Xss16m
-XX:+UseConcMarkSweepGC
-XX:+CMSParallelRemarkEnabled
-XX:ConcGCThreads=4
-XX:ReservedCodeCacheSize=1024m
-XX:+AlwaysPreTouch
-XX:+TieredCompilation
-XX:+UseCompressedOops
-XX:SoftRefLRUPolicyMSPerMB=50
-Dsun.io.useCanonCaches=false
-Djava.net.preferIPv4Stack=true
-Djsse.enableSNIExtension=false
-ea
-Dide.no.platform.update=true
```
