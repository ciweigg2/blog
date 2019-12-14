cover: http://ciwei2.cn-sh2.ufileos.com/4.jpg
title: Java中一行代码初始化集合
date: 2018-08-25 14:18:55
tags: [集合]
categories: [综合]
---
### Java中一行代码初始化集合
#### 1. 简介
在这个快速教程中，我们将研究如何使用一行代码初始化集合。
<!--more-->

#### 2. 利用数组创建
我们可以用一个数组创建集合，我们可以用数组工具类在一行程序中初始化它们：
```
    List<String> list = Arrays.asList(new String[]{"foo", "bar"});
```

我们能信任变量参数机制来处理数组的创建。这样，我们可以写出更简洁、更易读的代码:
```
@Test
public void givenArrayAsList_thenInitialiseList() {
    List<String> list = Arrays.asList("foo", "bar");

    assertTrue(list.contains("foo"));
}
```

该代码的运行结果是实现了集合的接口，但是它既不是java.util.ArrayList也不是LinkedList。相反，它是一个由原始数组支持的集合，该数组有两个含义。

尽管类的名称也叫ArrayList，但是它却在java.util.Arrays包中。

##### 2.1. 固定的长度
利用Arrays.asList创建的集合有固定的长度。
```
@Test(expected = UnsupportedOperationException.class)
public void givenArraysAsList_whenAdd_thenUnsupportedException() {
    List<String> list = Arrays.asList("foo", "bar");

    list.add("baz");
}
```

##### 2.2. 共享的引用
原始的数组和集合共享着对象的引用：
```
@Test
public void givenArraysAsList_whenCreated_thenShareReference(){
    String[] array = {"foo", "bar"};
    List<String> list = Arrays.asList(array);
    array[0] = "baz";

    assertEquals("baz", list.get(0));
}
```

#### 3. 利用流创建 (Java 8)
我们能很容易地将流转换为任何类型的集合。

因此，使用流的工厂方法，我们可以在一行程序中创建和初始化集合：
```
@Test
public void givenStream_thenInitializeList(){
    List<String> list = Stream.of("foo", "bar")
      .collect(Collectors.toList());

    assertTrue(list.contains("foo"));
}
```

在这里，我们应该注意Collectors.toList()并不能保证返回的集合是准确的。

对于返回的实例的可变性、可序列化性或线程安全性，一般没有约定。因此，我们的代码不应该依赖于这些属性。

一些人强调说，Stream.of(…).collect(…) 可能比Arrays.asList() 具有更大的内存和性能占用空间，但是在几乎所有情况下，它都是一个非常微小的优化，几乎没有什么区别。

#### 4. 工厂方法 (Java 9)
在JDK 9中，为集合引入了几种简便的工厂方法:
```
List<String> list = List.of("foo", "bar", "baz");
Set<String> set = Set.of("foo", "bar", "baz");
```
一个重要的细节是返回的实例是不可变的。此外，工厂方法在空间效率和线程安全方面有几个优势。

#### 5. 双括号初始化
在某些地方，我们发现一种名为“双括号初始化”的方法，它看起来如下:
```
@Test
public void givenAnonymousInnerClass_thenInitialiseList() {
    List<String> cities = new ArrayList() {{
        add("New York");
        add("Rio");
        add("Tokyo");
    }};

    assertTrue(cities.contains("New York"));
}
```

“双括号初始化”的名称很容易误导人。语法看起来紧凑而优雅，但它危险地隐藏了底层的内容。

实际上在Java中并没有“双括号”语法，它们是故意这样格式化的两个代码块。

使用外部大括号，我们声明一个匿名的内部类，它将是ArrayList的子类。在这些大括号中，我们可以声明子类的细节。

通常，我们可以使用实例初始化代码块，这也就是内部大括号的来源。

这种语法的简洁很吸引人，但它被认为是反模式的。
