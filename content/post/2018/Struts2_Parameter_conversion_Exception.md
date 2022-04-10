+++
title = "Struts2 参数转换异常处理"
date = 2018-03-18T19:51:00+08:00
lastmod = 2022-04-10T01:47:55+08:00
tags = ["Java", "Struts2"]
categories = ["Java"]
draft = false
toc = true
+++

## Struts2 参数转换异常处理 {#struts2-参数转换异常处理}

> 2018-03-18 19:51:33


### 异常描述 {#异常描述}

无法获取参数值,但也不报错, 只是警告 异常描述如下:

```text
18-Mar-2018 19:44:20.974 警告 [http-apr-8080-exec-1] com.opensymphony.xwork2.util.logging.jdk.JdkLogger.warn Error setting expression 'user.score' with value '[Ljava.lang.String;@6b961655'
```

User 类具体代码如下:

```java
package action.convert;

import java.util.Date;

/**
 * @author : Creeper
 * @date : 2018/3/18 15:53
 */
public class User {

    private String name;
    private int age;
    private double score;
    private Date birthday;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public double getscore() {
        return score;
    }

    public void setscore(double score) {
        this.score = score;
    }

    public Date getBirthday() {
        return birthday;
    }

    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", score=" + score +
                ", birthday=" + birthday +
                '}';
    }
}
```


#### 异常原因 {#异常原因}

因为 User 的 score 属性的 set,get 防范书写不规范, 所以导致获取不到值,
将方法名改为 `setScore` 和 `getScore` 即可
