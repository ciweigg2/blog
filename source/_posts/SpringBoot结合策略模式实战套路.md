title: SpringBoot结合策略模式实战套路
date: 2019-10-31 10:37:28
tags: [springboot,策略模式]
categories: [综合]
---
### 介绍

我们都知道设计模式好，可以让我们的代码更具可读性，扩展性，易于维护，但大部分程序猿一开始都学过至少一遍设计模式吧，实战中不知用到了几成。接下来让我介绍一个结合SpringBoot的策略模式套路，让你的代码少些if-else

<!--more-->

### 开撸

废话不多说，直接告诉你今天的核心是@autowired，看到这个是不是很熟悉，你每天都在用，不就是自动注入Spring管理的Bean吗？但我们对它的用法很多时候就局限在全局变量的注入了，忘记了，它其实还可以构造器注入，类型注入或命名注入，那么结合策略模式会绽放怎样的火花呢？跟着我的代码来看

#### 计算策略接口

```java
public interface CalculateStrategy {
    int doOperation(int num1,int num2);
}
```

#### 实现类

分别实现加减乘三个运算，可以看到我用了spring的注解@Component，也就是实例由spring管理了

```java
@Component
public class AddOperation implements CalculateStrategy {
    @Override
    public int doOperation(int num1, int num2) {
        return num1 + num2;
    }

}
```

```java
@Component
public class SubstractOperation implements CalculateStrategy {
    @Override
    public int doOperation(int num1, int num2) {
        return num1 - num2;
    }

}
```

```java
@Component
public class MultiplyOperation implements CalculateStrategy {
    @Override
    public int doOperation(int num1, int num2) {
        return num1 * num2;
    }

}
```

#### 上下文

之后创建上下文管理，用于提取策略。这个上下文才是本文的重点，注意到@autowired注解放的位置和对应的参数列表了吗？实际上它还可以注入到Map和List，Map的key就是它注入时的名，List则是存放全部实例对象

```java
import com.google.common.base.Preconditions;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import org.springframework.util.StringUtils;

import java.util.List;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

@Component
public class CalculatelOperationContext {

//    @Autowired
//    private Map<String, CalculateStrategy> strategyMap;

    private final Map<String, CalculateStrategy> strategyMap = new ConcurrentHashMap<>();

    @Autowired
    public void stragegyInteface(Map<String, CalculateStrategy> strategyMap) {
        this.strategyMap.clear();
        strategyMap.forEach(this.strategyMap::put);
        System.out.println(this.strategyMap);
    }


    @Autowired
    public void stragegyInteface2(List<CalculateStrategy> strategyMap) {
        strategyMap.forEach(System.out::println);
    }

    public CalculateStrategy strategySelect(String mode) {
        Preconditions.checkArgument(!StringUtils.isEmpty(mode), "不允许输入空字符串");
        return this.strategyMap.get(mode);
    }
}
```

打印结果：

```
{multiplyOperation=com.laoliang.springboot.pattern.strategy.MultiplyOperation@372ea2bc, addOperation=com.laoliang.springboot.pattern.strategy.AddOperation@4cc76301, substractOperation=com.laoliang.springboot.pattern.strategy.SubstractOperation@2f08c4b}
com.laoliang.springboot.pattern.strategy.AddOperation@4cc76301
com.laoliang.springboot.pattern.strategy.MultiplyOperation@372ea2bc
com.laoliang.springboot.pattern.strategy.SubstractOperation@2f08c4b
```

可以看到Map中key，value的关系，key的默认值为类的第一个字母小写

#### 控制层

```java
@RestController
public class StrategyController {

    @Autowired
    private CalculatelOperationContext calculatelOperationContext;

    @RequestMapping(value = "/operation")
    public String strategySelect(@RequestParam("mode") String mode) {
        return String.valueOf(calculatelOperationContext.strategySelect(mode).doOperation(20, 5));
    }
}
```

启动SpringBoot，浏览器调用http://localhost:8080/operation?mode=multiplyOperation，结果100。模式可以选择另外两个addOperation和substractOperation

我这里就做个演示，输入参数就写固定了，可以看到我们通过上下文calculatelOperationContext调用其方法strategySelect，通过不同的调用参数获得不同的策略，所以业务中只要可以抽象的方法都可以改写成这样的模式。

这种模式套路的好处就是当你要新增一种策略，比如除法，你不需要修改原来的代码，只要抽象不变，你新增一个DivideOperation类实现CalculateStrategy策略接口加上Spring注解即可，调用时模式修改为divideOperation就可以实现调用了，耦合性大大降低，不需要再改原来那一坨自己都看不下去的代码了

### 总结

可以看到全文中代码量还是相对比较少的，将不同的策略用不同的类实现，且可以不用改动别的代码，这篇文章你get到新套路了吗