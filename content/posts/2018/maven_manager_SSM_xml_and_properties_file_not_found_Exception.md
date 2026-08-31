+++
title = "maven管理SSM时xml和properties文件丢失异常"
date = 2017-12-28T18:52:00+08:00
lastmod = 2022-04-10T01:47:16+08:00
tags = ["Java", "Maven", "Exveption"]
categories = ["Java"]
draft = false
toc = true
+++

## classpath resource [com/qinkangdeid/mapping/] cannot be resolved to {#classpath-resource-com-qinkangdeid-mapping-cannot-be-resolved-to}

URL because it does not exist 和
org.apache.ibatis.binding.BindingException: Invalid bound statement (not
found)异常解决

:CUSTOM_ID: classpath-resource-comqinkangdeidmapping-cannot-be-resolved-to-url-because-it-does-not-exist-和-org.apache.ibatis.binding.bindingexception-invalid-bound-statement-not-found 异常解决

项目结构如图所示: ![](/images/post_images/20171228ResourceException.jpg)

```text
<build>
  <resources>
      <resource>
        <directory>src/main/java</directory>
          <includes>
            <include>**/*.properties</include>
            <include>**/*.xml</include>
          </includes>
          <filtering>false</filtering>
        </resource>
        <resource>
          <directory>src/main/resources</directory>
          <includes>
            <include>**/*.properties</include>
            <include>**/*.xml</include>
          </includes>
          <filtering>false</filtering>
        </resource>
      </resources>
</build>
```

如上图所示, 将 com.bobo.mapper 和 resource 目录配置到 pom 文件的 bulid 中即可
