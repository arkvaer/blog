+++
title = "JavaReview-day06-静态"
date = 2017-11-30T19:49:00+08:00
lastmod = 2022-04-10T01:45:36+08:00
tags = ["Java"]
categories = ["Java"]
draft = false
toc = true
+++

## Java 复习第 6 天 {#java 复习第 6 天}


### static 关键字 {#static-关键字}


#### static 的特点: {#static-的特点}

1.  内存特点

2.  存储位置: data segment(数据区 [也叫 共享区/方法区]);
3.  可以通过 "类名.静态成员" 的方式调用;
4.  随着类的加载而加载, 随着类的消失而消失(说明它的生命周期最长);
5.  优先于对象存在(加载);
6.  被所有对象所共享
7.  只有数据被多个对象共享是才适合定义静态
8.  可以直接被类名调用


#### 静态变量和非静态变量的举例 {#静态变量和非静态变量的举例}

```java
    class Person{
         String name;//成员变量，实例变量
       static String country = "CN";//静态变量。类变量
       public  void show(){

         System.out.println(Person.country+":"+this.name);

        }
      }
```

-   静态变量又叫类变量或静态成员变量, 非静态变量也叫成员变量或实例变量

-   静态变量和非静态变量的访问权限的不同
    -   static 方法只能访问 static 的变量，没有权限访问非 static 变量。
    -   static 方法中声明的变量可以与非 static 变量并且是类的属性重名
    -   方法中声明的变量可以与非 static 变量并且是类的属性重名
    -   非 static 方法中可以方访问 static 变量。
    -   static 方法可以访问 static 属性


#### 实例变量(静态变量)和类变量(非静态变量)的区别 {#实例变量静态变量和类变量非静态变量的区别}

<ol class="org-ol">
<li>存放位置:</li>

<li>类变量随着类的加载而存在于方法区中;</li>
<li>实例变量随着对象的建立而存在于堆内存中;</li>

<li value="2">生命周期:</li>

<li>类变量的生命周期最长,随着类的消失而消失;</li>
<li>实例变量随着对象的消失而消失;</li>
</ol>


#### 静态使用注意事项 {#静态使用注意事项}

-   静态方法只能访问静态变量成员,
    非静态方法既可以访问静态成员也可以访问非静态成员;
-   静态方法中不可以定义 this, super 关键字, 因为静态成员优先于对象存在,
    所以静态 方法中不可能存在 this;
-   主函数(main) 也是静态的;


### main()函数 {#main 函数}

<ol class="org-ol">
<li>主函数: 是一个特殊的函数; 作为程序的入口, 可以被 JVM 调用;</li>
<li>主函数的定义:</li>

<li>public: 代表着该函数访问权限是最大的;</li>
<li>static: 代表主函数随着类的加载就已经存在了;</li>
<li>void: 主函数没有具体的返回值;</li>
<li>main: 不是关键字, 但是是一个特殊的单词, 可以被 JVM 识别;</li>
<li>(String[] args): 函数的参数,参数类型是一个字符串类型的数组;</li>

<li value="3">主函数的格式是固定的: JVM 可以识别的;</li>

<li>JVM 在调用主函数时, 传入的参数是 new String[0];</li>

<li>可以在通过命令行运行时使用以下方式向 main 函数中传入参数:</li>
</ol>

```sh
$java MainDemo hehe enen oo
```

PS: 向 main 函数中传入 "hehe" "enen" "oo" 三个参数

```java
  class MainDemo
  {
      public static void main(String[] args) {
        String[] arr = {"hehe", "haha", "oo", "enen"}
        TestMain.main(arr);
      }
  }
  class TestMain{
    public static void main(String[] args) {
      for(int x=0; x<args.length; x++){
          System.out.println(args[x]);
      }
    }
  }
```


#### 静态的使用条件 {#静态的使用条件}

<!--list-separator-->

-  可以分为两部分(因为静态修饰的内容有成员变量和函数):

    <ol class="org-ol">
    <li>什么时候定义静态变量:</li>

    <li>当对象中出现共享的数据时, 该数据被静态所修饰;</li>
    <li>对象中的特有数据要定义成非静态, 存在于堆内存中;</li>

    <li value="2">什么时候定义静态方法:</li>

    <li>当功能内部没有访问到非静态数据(对象特有数据)时,
    那么该功能可以定义成静态的;</li>
    </ol>


#### JVM 的编译特点 {#jvm 的编译特点}

-   JVM 编译文件时会先查找当前类中对其他类的引用, 如果有其引用的
    **class 文件**,如果没有, 再查找与其同名的 **java 文件**,如果有, 先编译其
    java 文件, 然后在编译当前文件, 没有则报错;


#### 类加载的特点 {#类加载的特点}

-   只有用到类中的内容是, 类才被加载

<!--listend-->

```java
class Demo{
  System.out.println("a");
}
class TestDemo{
  public static void main(String[] args) {
    Demo demo = null;
  }

}
```

以上代码执行后没有运行结果, 因为 `TestDemo` 中并没有实际用到 `Demo`
中的内容


### Java 帮助文档 {#java-帮助文档}


#### Java 注释常用标签实例 {#java 注释常用标签实例}

-   () 引用其他类:
    ( 标签允许你引用其他类的文档)。javadoc 会在其生成的 HTML 文件中，用@see 标签链接到其他文档。
-   (a) package. class#member label
    该标签与@see 及其相似，只是它可以用于行内，并且是用”label”作为超链接文本而不用”See
    Also”
-   (a)
    该标签产生到文档根目录的相对路径，用于文档树页面的显示超链接
-   (a)
    该标签从当前这个类的最直接的基类中继承相关文档到当前的文档注释中
-   (a) 该标签格式如下： (a)
    version-information
    其中，"version-information"可以是任何你认为适合作为版本说明的重要信息，如果 javadoc 命令行使用了”-version”标记，那么久可以从生成的 HTML 文档中提取出版本信息
-   (a) 该标签的格式如下： (a)
    author-information
    其中，"author-information",望文生义你也知道，应该是你的名字，也可以包括电子邮件地址或者其他任何适宜的信息
-   (a) 该标签允许你指定程序代码最早使用的版本，你将会在 HTML
    java 文档中看到他被用来指定所用的 JDK 版本
-   (a) 该标签用于方法文档中，形式如下： (a)
    parameter-name description
    其中，parameter-name 是方法的参数列表中标识符
-   (a) 异常
-   (a)
    该标签用于指出一些旧特性已由改进的新特性所取代，建议用户不要使用这些旧特征。


#### Java 注释的使用顺序 {#java 注释的使用顺序}

1.  (a) (classes and interfaces only, required)
2.  (a) (classes and interfaces only, required. See
    footnote 1)
3.  (a) (methods and constructors only)
4.  (a) (methods only)
5.  (a) ((a) is a synonym added in Javadoc
    1.2)
6.  ()
7.  (a)
8.  (a) (or (a) or (a))
9.  (a) (see How and When To Deprecate APIs)


#### 注意 {#注意}

-   一个类中默认会有一个空参数的构造函数,
    这个默认构造函数的权限和所属类一致;
-   默认构造函数的权限是随着类的权限变化而变化的;


#### 静态代码块 {#静态代码块}

-   格式:

    ```java
      class{
        static{
            //静态代码块中的执行语句
        }
      }
    ```

-   特点: 随着类的加载而执行, **且只执行一次**;

-   作用: 用于给类初始化


#### 应用: 类中各个代码块之间的执行顺序 {#应用-类中各个代码块之间的执行顺序}

```java
  class StaticCode{

    StaticCode(){
      System.out.print("b ");
    }

    static{
      System.out.print("a ");
    }

    {
      System.out.print("c ");
    }

    StaticCode(int x){
      System.out.print("d ");
    }
  }
class StaticCodeDemo{
  public static void main(String[] args) {
    new StaticCode(4);
  }
}
```

-   以上代码的输出结果是: `a c d`

<!--list-separator-->

-  原因:

    -   静态代码块的作用是给\*类\*初始化的, 随着类的加载而执行, **且只执行一次**;
    -   构造代码块的作用是给\*对象(所有对象)\*初始化的,对象一建立就运行,
        而且优先于构造函数执行;
    -   构造函数的作用是给\*对应对象\*初始化

    参考链接:
    [Java中普通代码块，构造代码块，静态代码块区别及代码示例](https://www.cnblogs.com/sophine/p/3531282.html)


#### 总结 {#总结}

1.  虚拟机在首次加载 Java 类时，会对静态初始化块、静态成员变量、静态方法进行一次初始化
2.  只有在调用 new 方法时才会创建类的实例
3.  类实例创建过程：按照父子继承关系进行初始化，首先执行父类的初始化块部分，然后是父类的构造方法；再执行本类继承的子类的初始化块，最后是子类的构造方法
4.  类实例销毁时候，首先销毁子类部分，再销毁父类部分


### 对象的初始化过程 {#对象的初始化过程}

```java
class class Person
{
    private int age;
    private String name = "Jack";
    private static String country = "CN";

    Person(String name, int age){
      this.name = name;
      this.age = age;
    }

    public void speak()
    {
        System.out.println("name=" + this.name + ", age="+age);
    }
}

class  PersonDemo
{
    public static void main(String[] args)
    {
        Person p = new Person("Tom", 20);
    }

}
```

以上代码中 =Person p = new Person("Tom", 20)=执行时 Java
所完成的工作: 1. 在栈内存中创建变量 'p'; 2. 因为 "new" 用到了
"**Person.class**", 所以先找到"**Person.class**"文件并加载到 内存中 3.
执行该类中的 **static 代码块**, 给 "**Person 类**"进行初始化 4.
在堆内存中开辟空间, 并分配内存地址 5. 在内存中建立对象的特有属性,
并进行默认初始化 6. 对属性进行显示初始化 7.
对对象进行构造代码块初始化 8. 对对象进行对应的构造函数初始化 9.
将内存地址赋给栈内存中的 'p' 变量


#### 对象调用成员过程 {#对象调用成员过程}


#### 单例设计模式 {#单例设计模式}

<!--list-separator-->

-  例子

    下面是一个简单的单例设计模式的例子

    ```java
      class Single{

        private Single(){}
        private static Single s = new Single();
        private static Single getInstance(){
          return s;
        }
      }

      class SingleDemo{
        public static void main(String[] args) {
          Single s = Single.getInstance();
        }
      }
    ```

<!--list-separator-->

-  饿汉式

    ```java
      class Single{
        private Single(){}
        private static Single single = new Single();
        public static Single getInstance(){
          return single;
        }
      }
    ```


#### 懒汉式 {#懒汉式}

```java
  class Single{
    private Single(){}
    private static Single single = null;
    public static Single getInstance(){
      if (single == null) {
        //如果在此卡住, 则会出现多个对象
        single = new Single();
      }
      return single;
    }

  }
```

<!--list-separator-->

-  懒汉式和饿汉式的区别

    -   饿汉式不管调不调用, 都会先加载对象,
        而懒汉式则会在需要时才会进行加载对象

<!--list-separator-->

-  懒汉式最终解决方案

    ```java
      class Single{
        private Single(){}
        private static Single single = null;
        public static Single getInstance(){
          if (single == null) {
            synchronized(Single.class){
              if (single == null) {
                single = new Single();
              }
            }

          }
          return single;
        }

      }
    ```

<!--list-separator-->

-  注意!

    -   懒汉式加载会有安全性问题: 如果同时被多个对象同时调用, 则会出现多个对象
    -   开发用饿汉式
