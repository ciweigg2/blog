cover: http://ciwei2.cn-sh2.ufileos.com/53.jpg
title: JAVA8集合的用法
date: 2018-09-03 15:18:13
tags: [集合]
categories: [综合]
---
有很多新特性这边用具体代码演示：
<!--more-->

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User implements Serializable {

    private String id;

    private String name;

    public static void main(String[] args) {
        //初始化数据
        List<User> users = new ArrayList<User>() {{
            add(new User("1", "测试"));
            add(new User("1", "测试1"));
            add(new User("2", "测试2"));
            add(new User("1", "测试12"));
            add(new User("3", "测试9"));
        }};
        //根据id去重方法1
        List<User> unique = users.stream().collect(
                collectingAndThen(
                        toCollection(() -> new TreeSet<User>(comparing(User::getId))), ArrayList::new)
        );
        System.out.println(JSON.toJSONString(unique));

        //根据id去重方法2
        List<User> news = new ArrayList<>();
        users.stream().filter(distinctByKey(p -> p.getId())).forEach(p -> news.add(p));
        System.out.println(JSON.toJSONString(news));

        //有一个字符串的list，要统计其中长度大于7的字符串的数量，用迭代来实现
        List<String> wordList = Arrays.asList("regular", "expression", "specified", "as", "a", "string", "must");
        //用Stream实现
        long countByStream = wordList.stream().filter(w -> w.length() > 7).count();
        System.out.println(countByStream);
        //显然，用stream实现更简洁，不仅如此，stream很容易实现并发操作，比如
        long countByParallelStream = wordList.parallelStream().filter(w -> w.length() > 7).count();
        System.out.println(countByParallelStream);

        //取出偶数
        List<Integer> list = Arrays.asList(1, 2, 3, 4);
        List<Integer> newList2 = list.stream().filter(x -> x % 2 == 0).collect(Collectors.toList());
        System.out.println(newList2);

        //过滤空字符串
        List<String> strings = Arrays.asList("abc", "", "bc", "efg", "abcd", "", "jkl");
        List<String> filtered = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.toList());
        System.out.println(filtered);

        //循环
        List<String> list2 = Arrays.asList("a", "ad", "dr");
        list2.stream().forEach(System.out::println);
        //或者如下也可以
        list2.stream().forEach(a -> System.out.println(a));
        //或者不创建流也可以直接使用函数
        list2.forEach(System.out::println);
        //或者
        list2.forEach(a -> System.out.println(a));


        //主要用来对传入的参数进行逻辑处理
        //用数组来转换集合
        List<Integer> list3 = Arrays.asList(9,3,3);
        //distinct()函数，是去重复函数
        list3 = list.stream().distinct().map(i -> i*i).collect(Collectors.toList());
        //打印输出list
        list3.forEach(System.out::println);

        //Collectors函数 可以集合成所需要的集合类型 以逗号分隔拼接
        List<String> list4 =Arrays.asList("asd","dsg");
        //把list集合转换成带逗号“，”的字符串
        String str=list4.stream().filter(a -> !a.isEmpty()).collect(Collectors.joining(","));
        System.out.println(str);
        //把得到的字符串转换为了数组了
        String[] split = str.split(",");
        System.out.println(JSON.toJSON(split));

        //统计函数 用来统计数组集合的最大最小平均总和的各个值
        List<Integer> list5 = Arrays.asList(12,34,23,15,3,36);
        IntSummaryStatistics stats= list5.stream().mapToInt(x -> x).summaryStatistics();
        //最大值
        System.out.println(stats.getMax());
        //最小值
        System.out.println(stats.getMin());
        //平均值
        System.out.println(stats.getAverage());
        //总数
        System.out.println(stats.getCount());
        //总和
        System.out.println(stats.getSum());
    }

    /**
     * 去重方法
     *
     * @param keyExtractor
     * @param <T>
     * @return
     */
    public static <T> Predicate<T> distinctByKey(Function<? super T, Object> keyExtractor) {
        Map<Object, Boolean> map = new ConcurrentHashMap<>();
        return t -> map.putIfAbsent(keyExtractor.apply(t), Boolean.TRUE) == null;
    }
```