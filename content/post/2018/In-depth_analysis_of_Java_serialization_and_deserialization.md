+++
title = "深入分析Java的序列化与反序列化"
date = 2018-02-08T21:33:00+08:00
lastmod = 2022-04-10T01:47:01+08:00
tags = ["Java"]
categories = ["Java"]
draft = false
toc = true
+++

### JavaAPI 中的定义: {#javaapi 中的定义}

`public interface Serializable` 类通过实现 `java.io.Serializable`
接口以启用其序列化功能。未实现此接口的类将无法使其任何状态序列化或反序列化。可序列化类的所有子类型本身都是可序列化的。序列化接口没有方法或字段，仅用于标识可序列化的语义。

要允许不可序列化类的子类型序列化，可以假定该子类型负责保存和恢复超类型的公用
`(public)`. 受保护的 `(protected)` 和（如果可访问）包 `(package)`
字段的状态。仅在子类型扩展的类有一个可访问的无参数构造方法来初始化该类的状态时，才可以假定子类型有此职责。如果不是这种情况，则声明一个类为可序列化类是错误的。该错误将在运行时检测到。

在反序列化过程中，将使用该类的公用或受保护的无参数构造方法初始化不可序列化类的字段。可序列化的子类必须能够访问无参数构造方法。可序列化子类的字段将从该流中恢复。

当遍历一个图形时，可能会遇到不支持 Serializable
接口的对象。在此情况下，将抛出
=NotSerializableException=，并将标识不可序列化对象的类。

在序列化和反序列化过程中需要特殊处理的类必须使用下列准确签名来实现特殊方法：

```java
 private void writeObject(java.io.ObjectOutputStream out)
     throws IOException
 private void readObject(java.io.ObjectInputStream in)
     throws IOException, ClassNotFoundException;
 private void readObjectNoData()
     throws ObjectStreamException;
```

`writeObject` 方法负责写入特定类的对象的状态，以便相应的 readObject
方法可以恢复它。通过调用 `out.defaultWriteObject` 可以调用保存 `Object`
的字段的默认机制。该方法本身不需要涉及属于其超类或子类的状态。通过使用
`writeObject` 方法或使用 `DataOutput`
支持的用于基本数据类型的方法将各个字段写入
=ObjectOutputStream=，状态可以被保存。

`readObject` 方法负责从流中读取并恢复类字段。它可以调用
`in.defaultReadObject`
来调用默认机制，以恢复对象的非静态和非瞬态字段。=defaultReadObject=方法使用流中的信息来分配流中通过当前对象中相应指定字段保存的对象的字段。这用于处理类演化后需要添加新字段的情形。该方法本身不需要涉及属于其超类或子类的状态。通过使用
writeObject 方法或使用 `DataOutput`
支持的用于基本数据类型的方法将各个字段写入
=ObjectOutputStream=，状态可以被保存。

在序列化流不列出给定类作为将被反序列化对象的超类的情况下，=readObjectNoData=
方法负责初始化特定类的对象状态。这在接收方使用的反序列化实例类的版本不同于发送方，并且接收者版本扩展的类不是发送者版本扩展的类时发生。在序列化流已经被篡改时也将发生；因此，不管源流是“敌意的”还是不完整的，=readObjectNoData=
方法都可以用来正确地初始化反序列化的对象。

将对象写入流时需要指定要使用的替代对象的可序列化类，应使用准确的签名来实现此特殊方法：

`ANY-ACCESS-MODIFIER Object writeReplace() throws ObjectStreamException;`

此 `writeReplace`
方法将由序列化调用，前提是如果此方法存在，而且它可以通过被序列化对象的类中定义的一个方法访问。因此，该方法可以拥有私有
`(private)`. 受保护的 (protected) 和包私有 `(package-private)`
访问。子类对此方法的访问遵循 java 访问规则。

在从流中读取类的一个实例时需要指定替代的类应使用的准确签名来实现此特殊方法。

`ANY-ACCESS-MODIFIER Object readResolve() throws ObjectStreamException;`

此 `readResolve` 方法遵循与 `writeReplace` 相同的调用规则和访问规则。

序列化运行时使用一个称为 `serialVersionUID`
的版本号与每个可序列化类相关联，该序列号在反序列化过程中用于验证序列化对象的发送者和接收者是否为该对象加载了与序列化兼容的类。如果接收者加载的该对象的类的
`serialVersionUID` 与对应的发送者的类的版本号不同，则反序列化将会导致
`InvalidClassException=。可序列化类可以通过声明名为 "=serialVersionUID`"
的字段（该字段必须是静态 `(static)`. 最终 `(final)` 的 long
型字段）显式声明其自己的 =serialVersionUID=：

`ANY-ACCESS-MODIFIER static final long serialVersionUID = 42L;`

如果可序列化类未显式声明
`serialVersionUID=，则序列化运行时将基于该类的各个方面计算该类的默认
=serialVersionUID=值，如“Java(TM) 对象序列化规范”中所述。不过，强烈建议
所有可序列化类都显式声明 =serialVersionUID` 值，原因是计算默认的
`serialVersionUID`
对类的详细信息具有较高的敏感性，根据编译器实现的不同可能千差万别，这样在反序列化过程中可能会导致意外的
`InvalidClassException=。因此，为保证 =serialVersionUID` 值跨不同 java
编译器实现的一致性，序列化类必须声明一个明确的 `serialVersionUID`
值。还强烈建议使用 `private` 修饰符显示声明
`serialVersionUID=（如果可能），原因是这种声明仅应用于直接声明类 --
=serialVersionUID` 字段作为继承成员没有用处。数组类不能声明一个明确的
`serialVersionUID=，因此它们总是具有默认的计算值，但是数组类没有匹配
=serialVersionUID` 值的要求。 从以下版本开始： `JDK1.1`


## 深入分析 Java 的序列化与反序列化 {#深入分析 java 的序列化与反序列化}

序列化是一种对象持久化的手段。普遍应用在网络传输.
RMI 等场景中。本文通过分析 ArrayList 的序列化来介绍 Java 序列化的相关内容。主要涉及到以下几个问题：

-   怎么实现 Java 的序列化

-   为什么实现了 java.io.Serializable 接口才能被序列化

-   transient 的作用是什么

-   怎么自定义序列化策略

-   自定义的序列化策略是如何被调用的

-   ArrayList 对序列化的实现有什么好处


### Java 对象的序列化 {#java 对象的序列化}

Java 平台允许我们在内存中创建可复用的 Java 对象，但一般情况下，只有当 JVM 处于运行时，这些对象才可能存在，即，这些对象的生命周期不会比 JVM 的生命周期更长。但在现实应用中，就可能要求在 JVM 停止运行之后能够保存(持久化)指定的对象，并在将来重新读取被保存的对象。Java 对象序列化就能够帮助我们实现该功能。

使用 Java 对象序列化，在保存对象时，会把其状态保存为一组字节，在未来，再将这些字节组装成对象。必须注意地是，对象序列化保存的是对象的”状态”，即它的成员变量。由此可知，对象序列化不会关注类中的静态变量。

除了在持久化对象时会用到对象序列化之外，当使用 RMI(远程方法调用)，或在网络中传递对象时，都会用到对象序列化。Java 序列化 API 为处理对象序列化提供了一个标准机制，该 API 简单易用。


### 如何对 Java 对象进行序列化与反序列化 {#如何对 java 对象进行序列化与反序列化}

在 Java 中，只要一个类实现了 java.io.Serializable 接口，那么它就可以被序列化。这里先来一段代码：

code 1 创建一个 User 类，用于序列化及反序列化

```java
package com.hollis;
import java.io.Serializable;
import java.util.Date;

/**
 * Created by hollis on 16/2/2.
 */
public class User implements Serializable{
    private String name;
    private int age;
    private Date birthday;
    private transient String gender;
    private static final long serialVersionUID = -6849794470754667710L;

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

    public Date getBirthday() {
        return birthday;
    }

    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }

    public String getGender() {
        return gender;
    }

    public void setGender(String gender) {
        this.gender = gender;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", gender=" + gender +
                ", birthday=" + birthday +
                '}';
    }
}
```

code 2 对 User 进行序列化及反序列化的 Demo

```java
package com.hollis;
import org.apache.commons.io.FileUtils;
import org.apache.commons.io.IOUtils;
import java.io.*;
import java.util.Date;

/**
 * Created by hollis on 16/2/2.
 */
public class SerializableDemo {

    public static void main(String[] args) {
        //Initializes The Object
        User user = new User();
        user.setName("hollis");
        user.setGender("male");
        user.setAge(23);
        user.setBirthday(new Date());
        System.out.println(user);

        //Write Obj to File
        ObjectOutputStream oos = null;
        try {
            oos = new ObjectOutputStream(new FileOutputStream("tempFile"));
            oos.writeObject(user);
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            IOUtils.closeQuietly(oos);
        }

        //Read Obj from File
        File file = new File("tempFile");
        ObjectInputStream ois = null;
        try {
            ois = new ObjectInputStream(new FileInputStream(file));
            User newUser = (User) ois.readObject();
            System.out.println(newUser);
        } catch (IOException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } finally {
            IOUtils.closeQuietly(ois);
            try {
                FileUtils.forceDelete(file);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }

    }
}
```

output

```java
User{name='hollis', age=23, gender=male, birthday=Tue Feb 02 17:37:38 CST 2016}
User{name='hollis', age=23, gender=null, birthday=Tue Feb 02 17:37:38 CST 2016}
```


### 序列化及反序列化相关知识 {#序列化及反序列化相关知识}

1.  在 Java 中，只要一个类实现了 java.io.Serializable 接口，那么它就可以被序列化。

2.  通过 ObjectOutputStream 和 ObjectInputStream 对对象进行序列化及反序列化

3.  虚拟机是否允许反序列化，不仅取决于类路径和功能代码是否一致，一个非常重要的一点是两个类的序列化
    ID 是否一致（就是 private static final long serialVersionUID）

4.  序列化并不保存静态变量。

5.  要想将父类对象也序列化，就需要让父类也实现 Serializable 接口。

6.  Transient
    关键字的作用是控制变量的序列化，在变量声明前加上该关键字，可以阻止该变量被序列化到文件中，在被反序列化后，transient
    变量的值被设为初始值，如 int 型的是 0，对象型的是 null。

7.  服务器端给客户端发送序列化对象数据，对象中有一些数据是敏感的，比如密码字符串等，希望对该密码字段在序列化时，进行加密，而客户端如果拥有解密的密钥，只有在客户端进行反序列化时，才可以对密码进行读取，这样可以一定程度保证序列化对象的数据安全。


### ArrayList 的序列化 {#arraylist 的序列化}

在介绍 ArrayList 序列化之前，先来考虑一个问题：

如何自定义的序列化和反序列化策略

带着这个问题，我们来看 java.util.ArrayList 的源码

code 3

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    private static final long serialVersionUID = 8683452581122892189L;
    transient Object[] elementData; // non-private to simplify nested class access
    private int size;
}
```

笔者省略了其他成员变量，从上面的代码中可以知道=ArrayList=实现了=java.io.Serializable=接口，那么我们就可以对它进行序列化及反序列化。因为=elementData=是=transient=的，所以我们认为这个成员变量不会被序列化而保留下来。我们写一个 Demo，验证一下我们的想法：

code 4

```java
public static void main(String[] args) throws IOException, ClassNotFoundException {
        List<String> stringList = new ArrayList<String>();
        stringList.add("hello");
        stringList.add("world");
        stringList.add("hollis");
        stringList.add("chuang");
        System.out.println("init StringList" + stringList);
        ObjectOutputStream objectOutputStream = new ObjectOutputStream(new FileOutputStream("stringlist"));
        objectOutputStream.writeObject(stringList);

        IOUtils.close(objectOutputStream);
        File file = new File("stringlist");
        ObjectInputStream objectInputStream = new ObjectInputStream(new FileInputStream(file));
        List<String> newStringList = (List<String>)objectInputStream.readObject();
        IOUtils.close(objectInputStream);
        if(file.exists()){
            file.delete();
        }
        System.out.println("new StringList" + newStringList);
    }
```

输出

```text
init StringList[hello, world, hollis, chuang]
new StringList[hello, world, hollis, chuang]
```

了解=ArrayList=的人都知道，=ArrayList=底层是通过数组实现的。那么数组=elementData=其实就是用来保存列表中的元素的。通过该属性的声明方式我们知道，他是无法通过序列化持久化下来的。那么为什么
**code 4** 的结果却通过序列化和反序列化把 List 中的元素保留下来了呢？


#### writeObject 和 readObject 方法 {#writeobject 和 readobject 方法}

在 ArrayList 中定义了来个方法： writeObject 和 readObject。

这里先给出结论:

在序列化过程中，如果被序列化的类中定义了 writeObject 和 readObject
方法，虚拟机会试图调用对象类里的 writeObject 和 readObject
方法，进行用户自定义的序列化和反序列化。

如果没有这样的方法，则默认调用是 ObjectOutputStream 的
defaultWriteObject 方法以及 ObjectInputStream 的 defaultReadObject
方法。

用户自定义的 writeObject 和 readObject
方法可以允许用户控制序列化的过程，比如可以在序列化的过程中动态改变序列化的数值。

来看一下这两个方法的具体实现：

code 5

```java
private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        elementData = EMPTY_ELEMENTDATA;

        // Read in size, and any hidden stuff
        s.defaultReadObject();

        // Read in capacity
        s.readInt(); // ignored

        if (size > 0) {
            // be like clone(), allocate array based upon size not capacity
            ensureCapacityInternal(size);

            Object[] a = elementData;
            // Read in all elements in the proper order.
            for (int i=0; i<size; i++) {
                a[i] = s.readObject();
            }
        }
    }
```

code 6

```java
private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException{
        // Write out element count, and any hidden stuff
        int expectedModCount = modCount;
        s.defaultWriteObject();

        // Write out size as capacity for behavioural compatibility with clone()
        s.writeInt(size);

        // Write out all elements in the proper order.
        for (int i=0; i<size; i++) {
            s.writeObject(elementData[i]);
        }

        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
    }
```

那么为什么 ArrayList 要用这种方式来实现序列化呢？


#### why transient {#why-transient}

ArrayList 实际上是动态数组，每次在放满以后自动增长设定的长度值，如果数组自动增长长度设为 100，而实际只放了一个元素，那就会序列化 99 个 null 元素。为了保证在序列化的时候不会将这么多 null 同时进行序列化，ArrayList 把元素数组设置为 transient。


#### why writeObject and readObject {#why-writeobject-and-readobject}

前面说过，为了防止一个包含大量空对象的数组被序列化，为了优化存储，所以，ArrayList 使用 transient 来声明 elementData。
但是，作为一个集合，在序列化过程中还必须保证其中的元素可以被持久化下来，所以，通过重写 writeObject
和 readObject 方法的方式把其中的元素保留下来。

writeObject 方法把 elementData 数组中的元素遍历的保存到输出流（ObjectOutputStream）中。

readObject 方法从输入流（ObjectInputStream）中读出对象并保存赋值到 elementData 数组中。

至此，我们先试着来回答刚刚提出的问题：

如何自定义的序列化和反序列化策略

答：可以通过在被序列化的类中增加 writeObject 和
readObject 方法。那么问题又来了：

虽然 ArrayList 中写了 writeObject 和 readObject
方法，但是这两个方法并没有显示的被调用啊。

那么如果一个类中包含 writeObject 和 readObject
方法，那么这两个方法是怎么被调用的呢?


### ObjectOutputStream {#objectoutputstream}

从 code
4 中，我们可以看出，对象的序列化过程通过 ObjectOutputStream 和 ObjectInputputStream 来实现的，那么带着刚刚的问题，我们来分析一下 ArrayList 中的 writeObject
和 readObject 方法到底是如何被调用的呢？

为了节省篇幅，这里给出 ObjectOutputStream 的 writeObject 的调用栈：

writeObject ---&gt; writeObject0
---&gt;writeOrdinaryObject---&gt;writeSerialData---&gt;invokeWriteObject

这里看一下 invokeWriteObject：

````java
void invokeWriteObject(Object obj, ObjectOutputStream out)
        throws IOException, UnsupportedOperationException
    {
        if (writeObjectMethod != null) {
            try {
                writeObjectMethod.invoke(obj, new Object[]{ out });
            } catch (InvocationTargetException ex) {
                Throwable th = ex.getTargetException();
                if (th instanceof IOException) {
                    throw (IOException) th;
                } else {
                    throwMiscException(th);
                }
            } catch (IllegalAccessException ex) {
                // should not occur, as access checks have been suppressed
                throw new InternalError(ex);
            }
        } else {
            throw new UnsupportedOperationException();
        }
    }
    ```
其中writeObjectMethod.invoke(obj, new Object[]{ out });是关键，通过反射的方式调用writeObjectMethod方法。官方是这么解释这个writeObjectMethod的：

class-defined writeObject method, or null if none

在我们的例子中，这个方法就是我们在ArrayList中定义的writeObject方法。通过反射的方式被调用了。

至此，我们先试着来回答刚刚提出的问题：

如果一个类中包含writeObject 和 readObject 方法，那么这两个方法是怎么被调用的?

答：在使用ObjectOutputStream的writeObject方法和ObjectInputStream的readObject方法时，会通过反射的方式调用。

至此，我们已经介绍完了ArrayList的序列化方式。那么，不知道有没有人提出这样的疑问：

Serializable明明就是一个空的接口，它是怎么保证只有实现了该接口的方法才能进行序列化与反序列化的呢？

### Serializable接口的定义：

```java
public interface Serializable {
}
````

读者可以尝试把 **code 1** 中的继承=Serializable=的代码去掉，再执行 **code
2** ，会抛出=java.io.NotSerializableException=。

其实这个问题也很好回答，我们再回到刚刚 ObjectOutputStream 的 writeObject 的调用栈：
writeObject ---&gt; writeObject0
---&gt;writeOrdinaryObject---&gt;writeSerialData---&gt;invokeWriteObject
=writeObject0=方法中有这么一段代码：

````java
if (obj instanceof String) {
        writeString((String) obj, unshared);
    } else if (cl.isArray()) {
        writeArray(obj, desc, unshared);
    } else if (obj instanceof Enum) {
        writeEnum((Enum<?>) obj, desc, unshared);
    } else if (obj instanceof Serializable) {
        writeOrdinaryObject(obj, desc, unshared);
    } else {
        if (extendedDebugInfo) {
            throw new NotSerializableException(
                cl.getName() + "\n" + debugInfoStack.toString());
        } else {
            throw new NotSerializableException(cl.getName());
        }
    }
````

在进行序列化操作时，会判断要被序列化的类是否是 Enum.
Array 和 Serializable 类型，如果不是则直接抛出=NotSerializableException=。


### 总结 {#总结}

1.  如果一个类想被序列化，需要实现=Serializable=接口。否则将抛出=NotSerializableException=异常，这是因为，在序列化操作过程中会对类型进行检查，要求被序列化的类必须属于 Enum.
    Array 和 Serializable 类型其中的任何一种。

2.  在变量声明前加上该关键字，可以阻止该变量被序列化到文件中。

3.  在类中增加=writeObject= 和 `readObject` 方法可以实现自定义序列化策略

> 本文转载自[深入分析Java的序列化与反序列化](http://www.hollischuang.com/archives/1140)
