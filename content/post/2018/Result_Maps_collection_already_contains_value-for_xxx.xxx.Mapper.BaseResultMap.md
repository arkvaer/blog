+++
title = "Result Maps collection already contains value for xxx.xxx.Mapper.BaseResultMap异常解决"
date = 2017-12-29T20:02:00+08:00
lastmod = 2022-04-10T01:47:46+08:00
tags = ["Java", "MyBatis"]
draft = false
toc = true
+++

## Result Maps collection already contains value for xxx.xxx.Mapper.BaseResultMap 异常解决 {#result-maps-collection-already-contains-value-for-xxx-dot-xxx-dot-mapper-dot-baseresultmap-异常解决}

突然项目启动时报出以下错误: 百度上的所有答案没能解决,
最终索性重新把 mapper 文件重新生成了一遍, 结果令人感到意外, 竟然不报错了,
至于原因好像是运行过一个 mybatis-gen

```text
Exception sending context initialized event to listener instance of class org.springframework.web.context.ContextLoaderListener
org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'userServiceImpl': Injection of autowired dependencies failed; nested exception is org.springframework.beans.factory.BeanCreationException: Could not autowire field: private com.bobo.mapper.UserMapper com.bobo.service.impl.UserServiceImpl.userMapper; nested exception is org.springframework.beans.factory.NoSuchBeanDefinitionException: No qualifying bean of type [com.bobo.mapper.UserMapper] found for dependency: expected at least 1 bean which qualifies as autowire candidate for this dependency. Dependency annotations: {@org.springframework.beans.factory.annotation.Autowired(required=true)}
Related cause: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'userMapper' defined in file [F:\Work\IDEA\bobo\target\classes\com\bobo\mapper\UserMapper.class]: Invocation of init method failed; nested exception is java.lang.IllegalArgumentException: org.apache.ibatis.builder.BuilderException: Error parsing Mapper XML. Cause: java.lang.IllegalArgumentException: Result Maps collection already contains value for com.bobo.mapper.UserMapper.BaseResultMap
Related cause: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'userMapper' defined in file [F:\Work\IDEA\bobo\target\classes\com\bobo\mapper\UserMapper.class]: Invocation of init method failed; nested exception is java.lang.IllegalArgumentException: org.apache.ibatis.builder.BuilderException: Error parsing Mapper XML. Cause: java.lang.IllegalArgumentException: Result Maps collection already contains value for com.bobo.mapper.UserMapper.BaseResultMap
Related cause: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'userMapper' defined in file [F:\Work\IDEA\bobo\target\classes\com\bobo\mapper\UserMapper.class]: Invocation of init method failed; nested exception is java.lang.IllegalArgumentException: org.apache.ibatis.builder.BuilderException: Error parsing Mapper XML. Cause: java.lang.IllegalArgumentException: Result Maps collection already contains value for com.bobo.mapper.UserMapper.BaseResultMap
Related cause: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'userMapper' defined in file [F:\Work\IDEA\bobo\target\classes\com\bobo\mapper\UserMapper.class]: Invocation of init method failed; nested exception is java.lang.IllegalArgumentException: org.apache.ibatis.builder.BuilderException: Error parsing Mapper XML. Cause: java.lang.IllegalArgumentException: Result Maps collection already contains value for com.bobo.mapper.UserMapper.BaseResultMap
Related cause: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'avisMapper' defined in file [F:\Work\IDEA\bobo\target\classes\com\bobo\mapper\AvisMapper.class]: Invocation of init method failed; nested exception is java.lang.IllegalArgumentException: org.apache.ibatis.builder.BuilderException: Error parsing Mapper XML. Cause: java.lang.IllegalArgumentException: Result Maps collection already contains value for com.bobo.mapper.AvisMapper.BaseResultMap
Related cause: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'avisMapper' defined in file [F:\Work\IDEA\bobo\target\classes\com\bobo\mapper\AvisMapper.class]: Invocation of init method failed; nested exception is java.lang.IllegalArgumentException: org.apache.ibatis.builder.BuilderException: Error parsing Mapper XML. Cause: java.lang.IllegalArgumentException: Result Maps collection already contains value for com.bobo.mapper.AvisMapper.BaseResultMap
Related cause: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'userMapper' defined in file [F:\Work\IDEA\bobo\target\classes\com\bobo\mapper\UserMapper.class]: Invocation of init method failed; nested exception is java.lang.IllegalArgumentException: org.apache.ibatis.builder.BuilderException: Error parsing Mapper XML. Cause: java.lang.IllegalArgumentException: Result Maps collection already contains value for com.bobo.mapper.UserMapper.BaseResultMap
Related cause: org.springframework.beans.factory.BeanCreationException: Error
```
