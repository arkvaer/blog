+++
title = "Mybatis 异常: Error parsing Mapper XML. Cause:java.lang.IllegalArgumentException: Result Maps collection already"
date = 2018-03-10T17:31:00+08:00
lastmod = 2022-04-10T01:47:24+08:00
tags = ["Java", "Exception"]
categories = ["Java"]
draft = false
toc = true
+++

## Mybatis 异常: Error parsing Mapper XML. Cause: {#mybatis-异常-error-parsing-mapper-xml-dot-cause}

java.lang.IllegalArgumentException: Result Maps collection already

:CUSTOM_ID: mybatis-异常-error-parsing-mapper-xml.-cause-java.lang.illegalargumentexception-result-maps-collection-already

-   2018-03-10 17:31:39

> 前几天遇到一个非常神奇的错误,Error parsing Mapper XML. Cause:

java.lang.IllegalArgumentException: Result Maps collection already,
搞了半天,也没搞明白,
而且时有时无,很是神奇,最终发现最终原因是因为.......懒!


### 错误描述 {#错误描述}

```text
Error parsing Mapper XML. Cause: java.lang.IllegalArgumentException: Result Maps collection already contains
value for org.hc.jiankunking.system.member.dao.OrderLogMapper.BaseResultMap;
nestedexception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'sqlSessionFactory' defined in file [E:\16年文
件\JAVAdemo\jiankunking\jiankunking-web\target\jiankunking-web-1.0-SNAPSHOT\WEB-INF\classes\spring\spring-mybatis.xml]: Invocation of init method
failed; nested exception is org.springframework.core.NestedIOException:
Failed to parse mapping resource: 'URL [jar:file:/E:/16年文件/JAVAdemo/jiankunking/jiankunking-web/target/jiankunking-web-1.0-SNAPSHOT/
WEB-INF/lib/jiankunking-system-1.0-SNAPSHOT.jar!/mapper/OrderLogMapper.xml]'; nested exception is org.apache.ibatis.builder.BuilderException: Error
parsing Mapper XML. Cause: java.lang.IllegalArgumentException: Result Maps collection already contains value for
org.hc.jiankunking.system.member.dao.OrderLogMapper.BaseResultMap
```


### 解决方式 {#解决方式}

因为 Mybatis 官方提供了逆向工程的模块,
所以平时开发几乎都用这个,可以省不少事, 但是 问题来了,
在开发过程中有可能遇到数据库变动的情况, 所以这时就不得不重新生成 mapper
和 pojo 文件, 但是! 如果原先的文件不删除的话, 新生成的文件不会覆盖旧文件,
而是在旧 文件的结尾继续添加, 所以这也就是
`Result Maps collection already` 的原因了. 所以....这个懒还是不要偷的好,
重新生成文件前一定要把旧文件清理干净. 在这里给自己踩踩坑-_-!
