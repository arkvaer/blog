+++
title = "JavaReview-day13-String"
date = 2018-01-21T15:06:00+08:00
lastmod = 2022-04-10T01:46:00+08:00
draft = false
toc = true
+++

## String 及其他 {#string 及其他}

> 本篇主要介绍介绍 String 和其他基本数据类型的包装类型及其常用方法;

```java
class StringDemo
{
    public static void main(String[] args)
    {
        /*
        String s1 = "abc";//s1是一个类类型变量， "abc"是一个对象。
        //字符串最大特点：一旦被初始化就不可以被改变。

        String s2 = new String("abc");

        //s1和s2有什么区别？
        //s1在内存中有一个对象。
        //s2在内存中有两个对象。

        System.out.println(s1==s2);
        System.out.println(s1.equals(s2));//String类复写了Object类中equals方法，
        */
    //该方法用于判断字符串是否相同。

        String s = "abcde";
        method_1(s);
    }
    /*
    String类是对字符串事物的描述。
    该类定义了专门用于操作字符串的方法。
    "abc":
    */

    public static void method_1(String s)
    {
        char ch = s.charAt(3);

        System.out.println("ch="+ch);
        int num = s.codePointAt(3);

        System.out.println("num="+num);

        String s1 = "qq";
        s1 = s1.concat("mm");

        System.out.println("s1="+s1);
        System.out.println("qq"+"mm");

        String a = "opq";
        String b = "opq";
        System.out.println("a==b:"+(a==b));


    }
}
```


### String {#string}

String 类适用于描述字符串事物。 那么它就提供了多个方法对字符串进行操作。

常见的操作有哪些？ "abcd"


#### 1.获取。 {#获取}

```text
1. 字符串中的包含的字符数，也就是字符串的长度。
    int length():获取长度。
1. 根据位置获取位置上某个字符。
    char charAt(int index):
1. 根据字符获取该字符在字符串中位置。
    int indexOf(int ch):返回的是ch在字符串中第一次出现的位置。
    int indexOf(int ch, int fromIndex) :从fromIndex指定位置开始，获取ch在字符串中出现的位置。

    int indexOf(String str):返回的是str在字符串中第一次出现的位置。
    int indexOf(String str, int fromIndex) :从fromIndex指定位置开始，获取str在字符串中出现的位置。

    int lastIndexOf(int ch) ：
```


#### 2.判断。 {#判断}

```text
1. 字符串中是否包含某一个子串。
    boolean contains(str):
    特殊之处：indexOf(str):可以索引str第一次出现位置，如果返回-1.表示该str不在字符串中存在。
            所以，也可以用于对指定判断是否包含。
            if(str.indexOf("aa")!=-1)

            而且该方法即可以判断，有可以获取出现的位置。

2. 字符中是否有内容。
    boolean isEmpty(): 原理就是判断长度是否为0.
3. 字符串是否是以指定内容开头。
    boolean startsWith(str);
4. 字符串是否是以指定内容结尾。
    boolean endsWith(str);
5. 判断字符串内容是否相同。复写了Object类中的equals方法。
    boolean equals(str);
6. 判断内容是否相同，并忽略大小写。
    boolean equalsIgnoreCase();
```


#### 3.转换。 {#转换}

```text
1. 将字符数组转成字符串。
    构造函数：String(char[])
              String(char[],offset,count):将字符数组中的一部分转成字符串。

    静态方法：
            static String copyValueOf(char[]);
            static String copyValueOf(char[] data, int offset, int count)

            static String valueOf(char[]):


2. 将字符串转成字符数组。**
    char[] toCharArray():

3. 将字节数组转成字符串。
        String(byte[])
        String(byte[],offset,count):将字节数组中的一部分转成字符串。

4. 将字符串转成字节数组。
        byte[]  getBytes():
5. 将基本数据类型转成字符串。
    static String valueOf(int)
    static String valueOf(double)

    //3+"";//String.valueOf(3);

    特殊：字符串和字节数组在转换过程中，是可以指定编码表的。
```


#### 4.替换 {#替换}

```text
String replace(oldchar,newchar);
```


#### 5.切割 {#切割}

```text
String[] split(regex);
```


#### 6.子串。获取字符串中的一部分。 {#子串获取字符串中的一部分}

```text
String substring(begin);
String substring(begin,end);
```


#### 7.转换，去除空格，比较。 {#转换去除空格比较}

```text
1. 将字符串转成大写或则小写。
     String toUpperCase();
     String toLowerCase();

2. 将字符串两端的多个空格去除。
    String trim();

3. 对两个字符串进行自然顺序的比较。
    int compareTo(string);
```

```java
class  StringMethodDemo
{

    public static void method_7()
    {
        String s = "    Hello Java     ";
        sop(s.toLowerCase());
        sop(s.toUpperCase());
        sop(s.trim());

        String s1 = "a1c";
        String s2 = "aaa";

        sop(s1.compareTo(s2));
    }
    public static void method_sub()
    {
        String s = "abcdef";

        sop(s.substring(2));//从指定位置开始到结尾。如果角标不存在，会出现字符串角标越界异常。
        sop(s.substring(2,4));//包含头，不包含尾。s.substring(0,s.length());
    }

    public static void  method_split()
    {
        String s = "zhagnsa,lisi,wangwu";

        String[] arr  = s.split(",");

        for(int x = 0; x<arr.length; x++)
        {
            sop(arr[x]);
        }
    }

    public static void method_replace()
    {
        String s = "hello java";

        //String s1 = s.replace('q','n');//如果要替换的字符不存在，返回的还是原串。


        String s1 = s.replace("java","world");
        sop("s="+s);
        sop("s1="+s1);
    }

    public static void method_trans()
    {
        char[] arr = {'a','b','c','d','e','f'};

        String s= new String(arr,1,3);

        sop("s="+s);

        String s1 = "zxcvbnm";

        char[] chs = s1.toCharArray();

        for(int x=0; x<chs.length; x++)
        {
            sop("ch="+chs[x]);
        }
    }
    public static void method_is()
    {
        String str = "ArrayDemo.java";

        //判断文件名称是否是Array单词开头。
        sop(str.startsWith("Array"));
        //判断文件名称是否是.java的文件。
        sop(str.endsWith(".java"));
        //判断文件中是否包含Demo
        sop(str.contains(".java"));


    }


    public static void method_get()
    {
        String str = "abcdeakpf";

        //长度
        sop(str.length());

        //根据索引获取字符。
        sop(str.charAt(4));//当访问到字符串中不存在的角标时会发生StringIndexOutOfBoundsException。


        //根据字符获取索引
        sop(str.indexOf('m',3));//如果没有找到，返回-1.

        //反向索引一个字符出现位置。
        sop(str.lastIndexOf("a"));


    }
    public static void main(String[] args)
    {
        method_7();
//      method_trans();
//      method_is();
//      method_get();
        /*
        String s1 = "abc";
        String s2 = new String("abc");

        String s3 = "abc";
        System.out.println(s1==s2);
        System.out.println(s1==s3);
        */
    }

    public static void sop(Object obj)
    {
        System.out.println(obj);
    }


}
```


## String 实战 {#string 实战}


### 实战一 {#实战一}

-   模拟一个 trim 方法，去除字符串两端的空格。 思路：
    1.  判断字符串第一个位置是否是空格，如果是继续向下判断，直到不是空格为止。
        结尾处判断空格也是如此。
    2.  当开始和结尾都判断到不是空格时，就是要获取的字符串。
-   将一个字符串进行反转。将字符串中指定部分进行反转，"abcdefg";abfedcg
    思路：
    1.  曾经学习过对数组的元素进行反转。
    2.  将字符串变成数组，对数组反转。
    3.  将反转后的数组变成字符串。
    4.  只要将或反转的部分的开始和结束位置作为参数传递即可。

<!--listend-->

```java

class StringTest
{

    public static void sop(String str)
    {
        System.out.println(str);
    }
    public static void main(String[] args)
    {
        String s = "      ab cd      ";

        sop("("+s+")");
//      s = myTrim(s);
//      sop("("+s+")");

        sop("("+reverseString(s)+")");

    }


    //练习二：将字符串反转。
    /*
    思路：
    1，将字符串变成数组。
    2，对数组反转。
    3，将数组变成字符串。
    */

    public static String reverseString(String s,int start,int end)
    {
        //字符串变数组。
        char[] chs = s.toCharArray();

        //反转数组。
        reverse(chs,start,end);

        //将数组变成字符串。
        return new String(chs);
    }
    public static String reverseString(String s)
    {
        return reverseString(s,0,s.length());

    }

    private static void reverse(char[] arr,int x,int y)
    {
        for(int start=x,end=y-1; start<end ; start++,end--)
        {
            swap(arr,start,end);
        }
    }
    private static void swap(char[] arr,int x,int y)
    {
        char temp = arr[x];
        arr[x] = arr[y];
        arr[y] = temp;
    }

    //练习一，去除字符串两端空格。
    public static String myTrim(String str)
    {
        int start = 0,end = str.length()-1;

        while(start<=end && str.charAt(start)==' ')
            start++;

        while(start<=end && str.charAt(end)==' ')
            end--;

        return str.substring(start,end+1);
    }
}
```


### 实战二 {#实战二}

-   获取一个字符串在另一个字符串中出现的次数。 "abkkcdkkefkkskk"
    -   思路：

    -   定义个计数器。
    -   获取 kk 第一次出现的位置。
    -   从第一次出现位置后剩余的字符串中继续获取 kk 出现的位置。
        每获取一次就计数一次。
    -   当获取不到时，计数完成。

<!--listend-->

```java
class  StringTest2
{

    /*
    练习三。
    */

    public static int getSubCount(String str,String key)
    {
        int count = 0;
        int index = 0;

        while((index=str.indexOf(key))!=-1)
        {
            sop("str="+str);
            str = str.substring(index+key.length());

            count++;
        }
        return count;
    }

    /*
    练习三，方式二。

    */
    public static int getSubCount_2(String str,String key)
    {
        int count = 0;
        int index = 0;

        while((index= str.indexOf(key,index))!=-1)
        {
            sop("index="+index);
            index = index + key.length();

            count++;
        }
        return count;
    }

    public static void main(String[] args)
    {
        String str = "kkabkkcdkkefkks";

        ///sop("count====="+str.split("kk").length);不建议使用。

        sop("count="+getSubCount_2(str,"kk"));
    }

    public static void sop(String str)
    {
        System.out.println(str);
    }
}
```


### 实战三 {#实战三}

-   获取两个字符串中最大相同子串。第一个动作：将短的那个串进行长度一次递减的子串打印。
    "abcwerthelloyuiodef" "cvhellobnm" 思路： 1.
    将短的那个子串按照长度递减的方式获取到。 2.
    将每获取到的子串去长串中判断是否包含， 如果包含，已经找到。

<!--listend-->

```java
class  StringTest3
{
    /*
    练习四。
    */
    public static String getMaxSubString(String s1,String s2)
    {

        String max = "",min = "";

        max = (s1.length()>s2.length())?s1: s2;

        min = (max==s1)?s2: s1;

    //sop("max="+max+"...min="+min);
        for(int x=0; x<min.length(); x++)
        {
            for(int y=0,z=min.length()-x; z!=min.length()+1; y++,z++)
            {
                String temp = min.substring(y,z);

                sop(temp);
                if(max.contains(temp))//if(s1.indexOf(temp)!=-1)
                    return temp;
            }
        }
        return "";
    }


    public static void main(String[] args)
    {
        String s1 = "ab";
        String s2 = "cvhellobnm";
        sop(getMaxSubString(s2,s1));
    }

    public static void sop(String str)
    {
        System.out.println(str);
    }
}
```


### 实战四 {#实战四}

/\* 对字符串中字符进行自然顺序排序。

思路： 1，字符串变成字符数组。 2，对数组排序，选择，冒泡，Arrays.sort();
3，将排序后的数组变成字符串。 "vcz1bdAa+cs"--&gt;abccdsvz

_作业： "12 0 99 -7 30 4 100 13"
要求对字符串中的数值进行排序。生成一个数值从小到大新字符串。 "-7 0 4 12
13 30 99 100"_

```java
public class StringTest {

    public static void main(String[] args) {
        String str = "12 0 99 -7 30 4 100 13";
        System.out.println(sortString(str));
        }
    static String  sortString(String str){

        String[] string = str.split(" ");
        List<Integer> integer = new ArrayList<>();
        for (int i = 0; i < string.length; i ++) {
            integer.add(Integer.parseInt(string[i]));
        }
        sort(integer);
        String result = "";
        for (Integer integer1 : integer) {
            result += integer1.toString() + " ";
        }
        return result;
    }
}
```


## StringBuffer {#stringbuffer}

StringBuffer 是字符串缓冲区。

是一个容器。 ### 特点： 1. 长度是可变化的。 2.
可以字节操作多个数据类型。 3. 最终会通过 toString 方法变成字符串。

C create U update R read D delete

1.  存储。 StringBuffer append():将指定数据作为参数添加到已有数据结尾处。
    StringBuffer insert(index,数据):可以将数据插入到指定 index 位置。

2.  删除。 StringBuffer
    delete(start,end):删除缓冲区中的数据，包含 start，不包含 end。
    StringBuffer deleteCharAt(index):删除指定位置的字符。

3.  获取。 char charAt(int index) int indexOf(String str) int
    lastIndexOf(String str) int length() String substring(int start, int
    end)

4.  修改。 StringBuffer replace(start,end,string); void setCharAt(int
    index, char ch) ;

5.  反转。 StringBuffer reverse();

6.  将缓冲区中指定数据存储到指定字符数组中。 void getChars(int srcBegin,
    int srcEnd, char[] dst, int dstBegin)


## StringBuilder {#stringbuilder}


### API: {#api}

\`一个可变的字符序列。这个类提供了一个 API 兼容
StringBuffer，但无法保证同步。本课程是专为使用作为一个下降的替代
StringBuffer 在地方的字符串缓冲区被一个线程使用（一般情况下）。在可能的情况下，建议优先使用这类
StringBuffer 它将在大多数实现更快。
在 StringBuilder 的主营业务是 append 和 insert 方法的重载，以便接受任何数据类型。每个有效地将一个给定的数据到一个字符串，然后追加或插入字符串的字符的字符串生成器。的 append 方法总是说这些人物在生成器的结束；的 insert 方法添加字符在指定点。

例如，如果 z 指字符串生成器对象的当前内容是“start”，然后调用方法 z.append("le")会导致字符串生成器包含“startle”，而 z.insert(4,
"le")会改变字符串生成器包含“starlet”。

一般来说，如果某人是一个 StringBuilder 实例，然后 sb.append(x)具有相同的效果 sb.insert(sb.length(),
x)。

每一个字符串生成器都有一个容量。只要字符串生成器中包含的字符序列的长度不超过容量，就没有必要分配一个新的内部缓冲区。如果内部缓冲区溢出，则自动作出较大的。

StringBuilder 不安全的实例用于多个线程。如果需要同步然后建议 StringBuffer 可用。

除非另有说明，通过 null 争论这类构造函数或方法会导致一个 NullPointerException 被。\`

JDK1.5 版本之后出现了 StringBuilder.

**StringBuffer 是线程同步。 StringBuilder 是线程不同步。**

**以后开发，建议使用 StringBuilder**

Java 升级的三个因素： 1. 提高效率。 2. 简化书写。 3. 提高安全性。

```java
class Demo
{
}

public class StringBufferDemo
{
    public static void main(String[] args)
    {
        //method_update();

        StringBuilder sb = new StringBuilder("abcdef");

        char[] chs = new char[6];


        sb.getChars(1,4,chs,1);//将

        for(int x=0; x<chs.length; x++)
        {
            sop("chs["+x+"]="+chs[x]+";");
        }

        draw(3,6);
        draw(8,9);

//  StringBuilder sb1 = new StringBuilder();
//  sb1.append(new Demo()).append(new Demo());
//  sop("sb1="+sb1);
    }
    public static void method_update()
    {
        StringBuffer sb  = new StringBuffer("abcde");

//  sb.replace(1,4,"java");
        sb.setCharAt(2,'k');


        sop(sb.toString());

    }
    public static void method_del()
    {
        StringBuffer sb  = new StringBuffer("abcde");

//      sb.delete(1,3);
        //清空缓冲区。
        //sb.delete(0,sb.length());

        //sb.delete(2,3);
        sb.deleteCharAt(2);

        sop(sb.toString());
    }

    public static void method_add()
    {
        StringBuffer sb = new StringBuffer();


        //sb.append("abc").append(true).append(34);
//      StringBuffer sb1 = sb.append(34);
//      sop("sb==sb1:"+(sb==sb1));

        sb.insert(1,"qq");
        sop(sb.toString());//abctrue34
        //sop(sb1.toString());


    }


    public static void sop(String str)
    {
        System.out.println(str);
    }

    public static void draw(int row,int col)
    {
        StringBuilder sb = new StringBuilder();
        for(int x=0; x<row; x++)
        {
            for(int y=0; y<col; y++)
            {
                sb.append("*");
            }
            sb.append("\r\n");
        }

        sop(sb.toString());
    }

}
```


## 包装类 {#包装类}

/\* 基本数据类型对象包装类。 | 引用类型 | 包装类型 | |:-----:|:-----:| |
byte | Byte | | short | Short | | int | Integer | | long | Long | |
boolean | Boolean | | float | Float | | double | Double | | char |
Character |


### 基本数据类型对象包装类的最常见作用， {#基本数据类型对象包装类的最常见作用}

就是用于基本数据类型和字符串类型之间做转换

1.  基本数据类型转成字符串。

    基本数据类型+“”

    基本数据类型.toString(基本数据类型值);

    如： =Integer.toString(34);=//将 34 整数变成”34”;

2.  字符串转成基本数据类型。

    \`xxx a = Xxx.parseXxx(String);

    int a = Integer.parseInt("123");

    double b = Double.parseDouble("12.23");

    boolean b = Boolean.parseBoolean("true");

    Integer i = new Integer("123");

    int num = i.intValue();\`

3.  进制转换

4.  十进制转成其他进制。
    `toBinaryString();     toHexString();     toOctalString();`
5.  其他进制转成十进制。 `parseInt(string,radix);`

<!--listend-->

```java
class IntegerDemo
{
    public static void sop(String str)
    {
        System.out.println(str);
    }

    public static void main(String[] args)
    {
        //整数类型的最大值。
        //sop("int max :"+Integer.MAX_VALUE);

//      将一个字符串转成整数。

        int num = Integer.parseInt("123");//必须传入数字格式的字符串。
        //long x = Long.parseLong("123");

//      sop("num="+(num+4));

//      sop(Integer.toBinaryString(-6));
//      sop(Integer.toHexString(60));

        int x = Integer.parseInt("3c",16);

        sop("x="+x);


    }
}
```


### Integer 常用方法 {#integer 常用方法}

/\* JDK1.5 版本以后出现的新特性。

```java
class IntegerDemo1
{
    public static void main(String[] args)
    {

//      Integer x = new Integer(4);

        Integer x = 4;//自动装箱。//new Integer(4)
        x = x/* x.intValue() */ + 2;//x+2:x 进行自动拆箱。变成成了int类型。和2进行加法运算。
        //再将和进行装箱赋给x。

        Integer m = 128;
        Integer n = 128;

        sop("m==n:"+(m==n));

        Integer a = 127;
        Integer b = 127;

        sop("a==b:"+(a==b));//结果为true。因为a和b指向了同一个Integer对象。
       //因为当数值在byte范围内容，对于新特性，如果该数值已经存在，则不会在开辟新的空间。
    }

    public static void method()
    {
        Integer x = new Integer("123");

        Integer y = new Integer(123);

        sop("x==y:"+(x==y));
        sop("x.equals(y):"+x.equals(y));
    }

    public static void sop(String str)
    {
        System.out.println(str);
    }

}
```


#### Demo {#demo}

```java
class  Demo
{

    public static void sop(Object str)//Object str = new Integer(4);
    {
        System.out.println(str);
    }
    public static void main(String[] args)
    {

        sop(4);
//      int[] arr = {3,1,5,8,23,9};
//
//      System.out.println(toString(arr));
//      System.out.println(toString_2(arr));
    }
    public static String toString_2(int[] arr)
    {
        StringBuilder sb = new StringBuilder();

        sb.append("[");

        for(int x=0; x<arr.length; x++)
        {
            if(x!=arr.length-1)
                sb.append(arr[x]+", ");
            else
                sb.append(arr[x]+"}");

        }
        return  sb.toString();
    }

    public static String toString(int[] arr)
    {
        String str = "[";

        for(int x=0; x<arr.length; x++)
        {
            if(x!=arr.length-1)
                str += arr[x]+", ";
            else
                str += arr[x]+"]";
        }
        return str;
    }
}
```
