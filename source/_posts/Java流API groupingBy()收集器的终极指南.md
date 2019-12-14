cover: http://ciwei2.cn-sh2.ufileos.com/55.jpg
title: Java流API groupingBy()收集器的终极指南
date: 2018-10-04 20:49:49
tags: [Java流API使用groupingBy]
categories: [综合]
---
### 概览
简单地说，groupingBy()收集器提供了类似SQL中GROUP BY子句的功能，不过它需要Java流API才能使用。

为了使用groupingBy()收集器，我们必须指定一个用来执行分组操作的属性。这个属性值是通过一个函数式接口的实现来提供的——通常是传递一个Lambda表达式。

例如，如果我们想根据字符串长度来对字符串进行分组，那么可以通过将方法引用String::length获取到的字符串长度值传给groupingBy()收集器来实现：

<!--more-->

```java
List<String> strings = List.of("a", "bb", "cc", "ddd"); 
Map<Integer, List<String>> result = strings.stream() 
  .collect(groupingBy(String::length)); 
System.out.println(result); // {1=[a], 2=[bb, cc], 3=[ddd]}
```

但是，收集器本身能够做的，远不止如上所示的这么多。

### 分组到一个自定义Map

如果你提供一个自定义的Map实现类，就可以使用groupingBy()的重载方法来完成如下操作：

```java
List<String> strings = List.of("a", "bb", "cc", "ddd");
TreeMap<Integer, List<String>> result = strings.stream()
  .collect(groupingBy(String::length, TreeMap::new, toList()));
System.out.println(result); // {1=[a], 2=[bb, cc], 3=[ddd]}
```

### 提供一个自定义的下游集合

如果您需要将相同分组的元素存储到一个自定义集合中，那么可以通过toCollection()收集器来完成。

例如，如果您想将每个分组的元素都存放到TreeSet，那么可以使用groupingBy()的重载方法：

```java
groupingBy(String::length, toCollection(TreeSet::new))
```

当然，这里我们有一个完整的例子

```java
List<String> strings = List.of("a", "bb", "cc", "ddd");
Map<Integer, TreeSet<String>> result = strings.stream()
  .collect(groupingBy(String::length, toCollection(TreeSet::new)));
System.out.println(result); // {1=[a], 2=[bb, cc], 3=[ddd]}
```

### 分组和统计各分组的数量

如果您只是想知道各分组中元素的数量，那么只需要简单地使用counting()收集器即可：

```java
groupingBy(String::length, counting())
```

这里是一个完整的示例：

```java
List<String> strings = List.of("a", "bb", "cc", "ddd");
Map<Integer, Long> result = strings.stream()
  .collect(groupingBy(String::length, counting()));
System.out.println(result); // {1=1, 2=2, 3=1}
```

### 分组并将各分组的元素拼成字符串

如果你需要对元素进行分组并将各个分组的元素拼成字符串，那么可以通过使用joining()收集器来实现：

```java
groupingBy(String::length, joining(",", "[", "]"))
```

为了进一步了解这一点，请查看下面的示例：

```java
List<String> strings = List.of("a", "bb", "cc", "ddd");
Map<Integer, String> result = strings.stream()
  .collect(groupingBy(String::length, joining(",", "[", "]")));
System.out.println(result); // {1=[a], 2=[bb,cc], 3=[ddd]}
```

### 分组和过滤

有时，可能需要从分组结果中排除掉某些特定的元素。这种操作可以通过filtering()收集器来实现：

```java
groupingBy(String::length, filtering(s -> !s.contains("c"), toList()))
```

下面是使用filtering()收集器的示例代码：

```java
List<String> strings = List.of("a", "bb", "cc", "ddd");
Map<Integer, List<String>> result = strings.stream()
  .collect(groupingBy(String::length, filtering(s -> !s.contains("c"), toList())));
System.out.println(result); // {1=[a], 2=[bb], 3=[ddd]
```

### 分组和计算各组的平均值

如果你需要计算各个分组的平均值，那么有几个方便的收集器可以使用：

* averagingInt()
* averagingLong()
* averagingDouble()

这些收集器的使用示例如下所示：

```java
List<String> strings = List.of("a", "bb", "cc", "ddd");
Map<Integer, Double> result = strings.stream()
  .collect(groupingBy(String::length, averagingInt(String::hashCode)));
System.out.println(result); // {1=97.0, 2=3152.0, 3=99300.0}
```

> 温馨提示：
String::hashCode仅仅是被用作占位符，大家需要根据实际需求进行替换。

### 分组和计算各组的和

如果你想对集合内的元素求和，这里也有一些可用的收集器：

* summingInt()
* summingLong()
* summingDouble()

这些收集器的使用示例如下所示：

```java
List<String> strings = List.of("a", "bb", "cc", "ddd");
Map<Integer, Integer> result = strings.stream()
  .collect(groupingBy(String::length, summingInt(String::hashCode)));
System.out.println(result); // {1=97, 2=6304, 3=99300}
```

> 温馨提示：
String::hashCode仅仅是被用作占位符，大家可以根据实际需求替换成合适的值。

### 分组并计算统计摘要

如果你想进行分组操作，然后获取各个分组的统计摘要，那么也有一些开箱即用的收集器：

* summarizingInt()
* summarizingLong()
* summarizingDouble()
这些收集器的使用示例如下：

```java
List<String> strings = List.of("a", "bb", "cc", "ddd");
Map<Integer, IntSummaryStatistics> result = strings.stream()
  .collect(groupingBy(String::length, summarizingInt(String::hashCode)));
System.out.println(result);
```

让我们看一下打印的输出结果：

```java
{
    1=IntSummaryStatistics{
      count=1, 
      sum=97, 
      min=97, 
      average=97.000000, 
      max=97}, 
    2=IntSummaryStatistics{
      count=2, 
      sum=6304, 
      min=3136, 
      average=3152.000000, 
      max=3168}, 
    3=IntSummaryStatistics{
      count=1, 
      sum=99300, 
      min=99300, 
      average=99300.000000, 
      max=99300}
}
```

> 温馨提示：
String::hashCode仅仅是被用作占位符，大家需要根据实际需求进行替换。

### 分组和归约

如果您想对分组的元素执行归约操作，可以使用reducing()收集器：

```java
groupingBy(List::size, reducing(List.of(), (l1, l2) -> ...)))
```

reducing()收集器的使用示例如下所示：

```java
List<String> strings = List.of("a", "bb", "cc", "ddd");
Map<Integer, List<Character>> result = strings.stream()
  .map(toStringList())
  .collect(groupingBy(List::size, reducing(List.of(), (l1, l2) -> Stream.concat(l1.stream(), l2.stream())
    .collect(Collectors.toList()))));
System.out.println(result); // {1=[a], 2=[b, b, c, c], 3=[d, d, d]}
```

###分组和获取最大/最小值

如果您想从分组中获取最大/最小的元素，那么只需要简单地使用max()/min()收集器就可以了：

```java
groupingBy(String::length, Collectors.maxBy(Comparator.comparing(String::toUpperCase)))
```

这些收集器的使用示例如下所示：

```java
List<String> strings = List.of("a", "bb", "cc", "ddd");
Map<Integer, Optional<String>> result = strings.stream()
  .collect(groupingBy(String::length, Collectors.maxBy(Comparator.comparing(String::toUpperCase))));
System.out.println(result); // {1=Optional[a], 2=Optional[cc], 3=Optional[ddd]}
```

在这种情况下，收集器返回一个Optional类型的数据通常会有点不方便。在一个分组中通常会不止一个元素，那么使用Optional类型的数据，通常只会增加额外的复杂度。

不幸的是，我们不能通过收集器来避免返回Optional类型的数据。不过，我们可以使用reducing()收集器来实现要想的功能

### 组合下游收集器

一旦我们开始组合多个收集器来定义复杂的下游分组操作，就可以发挥收集器的全部能力了。这些下游分组操作就是标准的流API流水线—只有你想不到的，没有做不到的：一切皆有可能。

#### 示例 #1

比方说我们有一个希望根据字符串长度进行分组的字符串列表，并且 不仅希望将各个分组的字符串转换成大写，而且还要过滤掉字符串长度小于1的元素，最后将各个分组中的字符串分别存放到TreeSet实例中。

我们可以很容易地做到：

```java
var result = strings.stream()
  .collect(
    groupingBy(String::length,
      mapping(String::toUpperCase,
        filtering(s -> s.length() > 1,
          toCollection(TreeSet::new)))));
//result
{1=[], 2=[BB, CC], 3=[DDD]}
```

#### 示例 #2

给定一个字符串列表，先通过字符串的长度进行分组，然后将同一个分组中的字符串放到一个列表中，接着将所获得列表内的每一个字符串都转换成流的内容， 只保留具有非零长度的不同元素，最终把列表中的字符串通过reduce收集器拼成了一个字符串。

我们也可以实现这个需求：

```java
var result = strings.stream()
  .collect(
    groupingBy(String::length,
      mapping(toStringList(),
        flatMapping(s -> s.stream().distinct(),
          filtering(s -> s.length() > 0,
            mapping(String::toUpperCase,
              reducing("", (s, s2) -> s + s2)))))
    ));
//result 
{1=A, 2=BC, 3=D}
```

### 源码

> 温馨提示：
运行示例代码需要JDK1.9或更高版本

```java
    <dependencies>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-engine</artifactId>
            <version>5.0.0</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.assertj</groupId>
            <artifactId>assertj-core</artifactId>
            <version>3.8.0</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
```

CompoundGroupingTest.java

```java
package com.pivovarit.grouping;

import org.junit.jupiter.api.Test;

import java.util.List;
import java.util.Map;
import java.util.TreeSet;
import java.util.function.Function;

import static java.util.stream.Collectors.filtering;
import static java.util.stream.Collectors.flatMapping;
import static java.util.stream.Collectors.groupingBy;
import static java.util.stream.Collectors.mapping;
import static java.util.stream.Collectors.reducing;
import static java.util.stream.Collectors.toCollection;
import static java.util.stream.Collectors.toList;

class CompoundGroupingTest {
    private List<String> strings = List.of("a", "bb", "cc", "ddd");

    @Test
    void example_1() {
        Map<Integer, TreeSet<String>> result = strings.stream()
          .collect(
            groupingBy(String::length,
              mapping(String::toUpperCase,
                filtering(s -> s.length() > 1,
                  toCollection(TreeSet::new)))));

        System.out.println(result);
    }

    @Test
    void example_2() {
        Map<Integer, String> result = strings.stream()
          .collect(
            groupingBy(String::length,
              mapping(toStringList(),
                flatMapping(s -> s.stream().distinct(),
                  filtering(s -> s.length() > 0,
                    mapping(String::toUpperCase,
                      reducing("", String::concat)))))));

        System.out.println(result);
    }

    private static Function<String, List<String>> toStringList() {
        return s -> s.chars()
          .mapToObj(c -> (char) c)
          .map(Object::toString)
          .collect(toList());
    }
}
```

SimpleGroupingTest.java

```java
package com.pivovarit.grouping;

import org.junit.jupiter.api.Test;

import java.util.Collection;
import java.util.Comparator;
import java.util.IntSummaryStatistics;
import java.util.List;
import java.util.Map;
import java.util.Optional;
import java.util.TreeMap;
import java.util.TreeSet;
import java.util.function.Function;
import java.util.stream.Collectors;
import java.util.stream.Stream;

import static java.util.stream.Collectors.averagingInt;
import static java.util.stream.Collectors.counting;
import static java.util.stream.Collectors.filtering;
import static java.util.stream.Collectors.flatMapping;
import static java.util.stream.Collectors.groupingBy;
import static java.util.stream.Collectors.joining;
import static java.util.stream.Collectors.reducing;
import static java.util.stream.Collectors.summarizingInt;
import static java.util.stream.Collectors.summingInt;
import static java.util.stream.Collectors.toCollection;
import static java.util.stream.Collectors.toList;

class SimpleGroupingTest {

    @Test
    void standardGrouping() {
        List<String> strings = List.of("a", "bb", "cc", "ddd");

        Map<Integer, List<String>> result = strings.stream()
          .collect(groupingBy(String::length));

        System.out.println(result);
    }

    @Test
    void customMapImplementation() {
        List<String> strings = List.of("a", "bb", "cc", "ddd");

        TreeMap<Integer, List<String>> result = strings.stream()
          .collect(groupingBy(String::length, TreeMap::new, toList()));

        System.out.println(result);
    }

    @Test
    void customAggregateImplementation() {
        List<String> strings = List.of("a", "bb", "cc", "ddd");

        Map<Integer, TreeSet<String>> result = strings.stream()
          .collect(groupingBy(String::length, toCollection(TreeSet::new)));

        System.out.println(result);
    }

    @Test
    void customAggregation_counting() {
        List<String> strings = List.of("a", "bb", "cc", "ddd");

        Map<Integer, Long> result = strings.stream()
          .collect(groupingBy(String::length, counting()));

        System.out.println(result);
    }

    @Test
    void customAggregation_filtering() {
        List<String> strings = List.of("a", "bb", "cc", "ddd");

        Map<Integer, List<String>> result = strings.stream()
          .collect(groupingBy(String::length, filtering(s -> !s.contains("c"), toList())));

        System.out.println(result);
    }

    @Test
    void customAggregation_joining() {
        List<String> strings = List.of("a", "bb", "cc", "ddd");

        Map<Integer, String> result = strings.stream()
          .collect(groupingBy(String::length, joining(",", "[", "]")));

        System.out.println(result);
    }

    @Test
    void customAggregation_averaging() {
        List<String> strings = List.of("a", "bb", "cc", "ddd");

        Map<Integer, Double> result = strings.stream()
          .collect(groupingBy(String::length, averagingInt(String::hashCode)));

        System.out.println(result);
    }

    @Test
    void customAggregation_summarizing() {
        List<String> strings = List.of("a", "bb", "cc", "ddd");

        Map<Integer, IntSummaryStatistics> result = strings.stream()
          .collect(groupingBy(String::length, summarizingInt(String::hashCode)));

        System.out.println(result);
    }

    @Test
    void customAggregation_flatmapping() {
        List<String> strings = List.of("a", "bb", "cc", "ddd");

        Map<Integer, List<Character>> result = strings.stream()
          .map(toStringList())
          .collect(groupingBy(List::size, flatMapping(Collection::stream, Collectors.toList())));

        System.out.println(result);
    }

    @Test
    void customAggregation_mapping() {
        List<String> strings = List.of("a", "bb", "cc", "ddd");

        Map<Integer, List<String>> result = strings.stream()
          .collect(groupingBy(String::length, Collectors.mapping(String::toUpperCase, Collectors.toList())));

        System.out.println(result);
    }

    @Test
    void customAggregation_reducing() {
        List<String> strings = List.of("a", "bb", "cc", "ddd");

        Map<Integer, List<Character>> result = strings.stream()
          .map(toStringList())
          .collect(groupingBy(List::size, reducing(List.of(), (l1, l2) -> Stream.concat(l1.stream(), l2.stream())
            .collect(Collectors.toList()))));

        System.out.println(result);
    }

    @Test
    void customAggregation_reducing_optional() {
        List<String> strings = List.of("a", "bb", "cc", "ddd");

        Map<Integer, Optional<List<Character>>> result = strings.stream()
          .map(toStringList())
          .collect(groupingBy(List::size, reducing((l1, l2) -> Stream.concat(l1.stream(), l2.stream()).collect(Collectors.toList()))));

        System.out.println(result);
    }

    @Test
    void customAggregation_summing() {
        List<String> strings = List.of("a", "bb", "cc", "ddd");

        Map<Integer, Integer> result = strings.stream()
          .collect(groupingBy(String::length, summingInt(String::hashCode)));

        System.out.println(result);
    }

    @Test
    void customAggregation_max() {
        List<String> strings = List.of("a", "bb", "cc", "ddd");

        Map<Integer, Optional<String>> result = strings.stream()
          .collect(groupingBy(String::length, Collectors.maxBy(Comparator.comparing(String::toUpperCase))));

        System.out.println(result);
    }

    @Test
    void customAggregation_min() {
        List<String> strings = List.of("a", "bb", "cc", "ddd");

        Map<Integer, Optional<String>> result = strings.stream()
          .collect(groupingBy(String::length, Collectors.minBy(Comparator.comparing(String::toUpperCase))));

        System.out.println(result);
    }

    private static Function<String, List<Character>> toStringList() {
        return s -> s.chars().mapToObj(c -> (char) c).collect(toList());
    }
}
```