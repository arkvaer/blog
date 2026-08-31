+++
title = "RandomAccess接口理解"
date = 2018-04-01T22:47:00+08:00
lastmod = 2022-04-10T01:47:41+08:00
tags = ["Java", "Source_Code"]
categories = ["Java"]
draft = false
toc = true
+++

## RandomAccess 接口理解 {#randomaccess-接口理解}

> 本文转自
> [Stick2It
> --- RandomAccess接口理解](https://blog.csdn.net/stick2it/article/details/53469910)

根据 javadoc 上面的的解释是：

RandomAccess
是一个标记接口，用于标明实现该接口的 List 支持快速随机访问，主要目的是使算法能够在随机和顺序访问的 list 中表现的更加高效。

我们可以简单的看下 Collections 下的 binarySearch 方法的源码:

```java
view plain copy
public static <T>
    int binarySearch(List<? extends Comparable<? super T>> list, T key) {
        if (list instanceof RandomAccess || list.size()<BINARYSEARCH_THRESHOLD)
            return Collections.indexedBinarySearch(list, key);
        else
            return Collections.iteratorBinarySearch(list, key);
    }
```

从源码中我们可以看到，在进行二分查找的时候，list 会先判断是否是 RandomAccess 也即是否实现了 RandomAccess 接口，接着在调用想用的二分查找算法来进行，（其中:
BINARYSEARCH_THRESHOLD
Collections 的一个常量（5000），它是二分查找的阀值。）如果实现了 RandomAccess 接口的 List，执行 indexedBinarySearch 方法，否则执行
iteratorBinarySearch 方法。

分别看下这两个方法的实现:


### indexedBinarySearch 方法 {#indexedbinarysearch-方法}

```java
view plain copy
private static <T>
    int indexedBinarySearch(List<? extends Comparable<? super T>> list, T key) {
        int low = 0;
        int high = list.size()-1;

        while (low <= high) {
            int mid = (low + high) >>> 1;
            Comparable<? super T> midVal = list.get(mid);
            int cmp = midVal.compareTo(key);

            if (cmp < 0)
                low = mid + 1;
            else if (cmp > 0)
                high = mid - 1;
            else
                return mid; // key found
        }
        return -(low + 1);  // key not found
    }
```

indexedBinarySearch 方法是直接通过 get 来访问元素

iteratorBinarySearch 方法:

```java
view plain copy
private static <T>
    int iteratorBinarySearch(List<? extends Comparable<? super T>> list, T key)
    {
        int low = 0;
        int high = list.size()-1;
        ListIterator<? extends Comparable<? super T>> i = list.listIterator();

        while (low <= high) {
            int mid = (low + high) >>> 1;
            Comparable<? super T> midVal = get(i, mid);
            int cmp = midVal.compareTo(key);

            if (cmp < 0)
                low = mid + 1;
            else if (cmp > 0)
                high = mid - 1;
            else
                return mid; // key found
        }
        return -(low + 1);  // key not found
    }
```

iteratorBinarySearch 中 ListIterator 来查找相应的元素

javadoc 中特别指出:

It is recognized that the distinction between random and sequential
access is often fuzzy. For example, some List implementations provide
asymptotically linear access times if they get huge, but constant access
times in practice. Such a Listimplementation should generally implement
this interface. As a rule of thumb, a List implementation should
implement this interface if, for typical instances of the class, this
loop:

```java
     for (int i=0, n=list.size(); i < n; i++)
         list.get(i);

runs faster than this loop:
     for (Iterator i=list.iterator(); i.hasNext(); )
         i.next();
```


#### 总结：实现 RandomAccess 接口的的 List 可以通过简单的 for 循环来访问数据比使用 iterator 访问来的高效快速 {#总结实现 randomaccess 接口的的 list 可以通过简单的 for 循环来访问数据比使用 iterator 访问来的高效快速}


#### 参考文档:[[<https://docs.oracle.com/javase/7/docs/api/java/util/RandomAccess.html>][]] {#参考文档}
