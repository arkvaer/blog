+++
title = "FTP storFile返回false 解决方法"
date = 2017-12-04T19:59:00+08:00
lastmod = 2022-04-10T01:43:28+08:00
tags = ["Java", "Desgin_Patterns"]
categories = ["Java"]
draft = false
toc = true
+++

[本文参考地址](https://www.cnblogs.com/xiangpiaopiao2011/archive/2012/02/28/2371679.html#undefined)


## 错误描述 {#错误描述}

-   自己搭建的 FTP 服务器总是在执行 FTPClient.storFile(); 方法时返回 false


## 解决方法 {#解决方法}

-   添加以下代码即可

    ```text
        ftpClient.setControlEncoding("UTF-8");
    ```
