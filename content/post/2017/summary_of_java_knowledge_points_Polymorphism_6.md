+++
title = "JavaReview-day08-多态"
date = 2017-12-26T13:30:00+08:00
lastmod = 2022-04-10T01:45:33+08:00
tags = ["Java"]
categories = ["Java"]
draft = false
toc = true
+++

## 多态 {#多态}


### 多态的特点 {#多态的特点}

<ol class="org-ol">
<li>在多态中, 成员函数的特点:</li>

<li>在编译时期: 参阅引用型变量所属的类中是否有调用的方法. 如果有,
编译通过; 如果没有, 编译失败</li>
<li>在运行时期: 参阅对象所属类的中是否有调用的方法</li>
<li>**简单总结就是:** 成员函数在多态调用时, 编译看左边, 运行看右边;
因为成员函数存在覆盖特性。</li>

<li value="2">在多态中, 成员变量的特点:</li>

<li>无论编译和运行, 都参考左边(引用型变量所属的类);</li>

<li value="3">在多态中, 静态成员函数的特点:</li>

<li>无论编译和运行, 都参考左边(引用型变量所属的类);</li>
<li>其实对于静态方法，是不需要对象的。直接用类名调用即可。</li>
</ol>

```java
class Fu
{
//  int num = 3;
    void show()
    {
        System.out.println("fu show");
    }

    static void method()
    {
        System.out.println("fu static method");
    }
}

class Zi extends Fu
{
//  int num = 4;
    void show()
    {
        System.out.println("zi show");
    }

    static void method()
    {
        System.out.println("zi static method");
    }
}



class  DuoTaiDemo3
{
    public static void main(String[] args)
    {
        Fu.method();
        Zi.method();
        Fu f = new Zi();//
//      f.method();
//      f.show();
//      System.out.println(f.num);


//      Zi z = new Zi();
//      System.out.println(z.num);
    }
}

```
