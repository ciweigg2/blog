title: 使用策略模式代替if else
date: 2019-08-23 16:37:40
tags: [策略模式]
categories: [综合]
---
### 介绍

参考：[业务代码中，太多 if else 怎么办？](https://mp.weixin.qq.com/s?__biz=MzUxOTc4NjEyMw==&mid=2247484743&idx=1&sn=6a6dbe43f76545e87c1bef29d21a440b&chksm=f9f51ea3ce8297b50c79d66c9c8164bc0abfc78db48aa4ce3c96798851be9dd7b74df2db1b19&mpshare=1&scene=1&srcid=08233xHle8ZC62R3LDo20Y0B&sharer_sharetime=1566531607851&sharer_shareid=4224c9dd2345a4afaf582d35213c51be&key=77072c6a12a707e84af2159f9737f7ad4f265f1b5d5e455d1654bf381974998585d23fa4efcab709ec635d8522fdcbfa260d7c2bb00c02b7630177640bbad7ad4416c088ae4f76f48ecc52a8d4173c57&ascene=1&uin=NjA3MDIyNzgw&devicetype=Windows+10&version=62060833&lang=zh_CN&pass_ticket=GKHM%2FKhOTDYC0ThlC2I5s7WBFzvSjFJKBPhwMcyErsk0JRQ7KO1iUdyxsgnwflCU)

<!--more-->

前段时间，我将公司系统中的批量审单的功能进行了重构，用到了java的并发编程进行异步化处理，数据库的乐观锁机制处理多线程并发更新数据。其中批量审单的业务处理涉及到多种任务类型，对应不同的业务方法进行处理，比如转仓，转快递，添加赠品，删除赠品，拆分订单，批量驳回，批量作废等等，其中就用到了策略模式

### if else模式

```java
if ("BATCH_CHANGE_WAREHOUSE".equals(taskType)) {
    //批量转仓逻辑
} else if ("BATCH_CHANGE_SHIPPING".equals(taskType)) {
    //批量转快递逻辑
} else if ("BATCH_REPLACE_ORDER_GOODS".equals(taskType)) {
    //批量替换订单商品逻辑
} else if ("BATCH_DELETE_ORDER_GOODS".equals(taskType)) {
    //批量删除订单商品逻辑
} else if ("BATCH_ADD_MEMO".equals(taskType)) {
    //批量添加备注逻辑
} else {
    //任务类型未知
    System.out.println("任务类型无法处理");
}
```

看起来，思路清晰，if,else分支也很清楚，但不觉得代码很臃肿，维护起来麻烦吗，尤其是其他人来接锅的时候，连看下去的欲望都没有了。这时候你需要用策略模式消除其中的if else，进行一下简单的重构

### 策略模式

#### 1、首先抽象业务处理器

```java
public abstract class InspectionSolver {

    public abstract void solve(Long orderId, Long userId);

    public abstract String[] supports();
}
```

#### 2、将业务处理器和其支持处理的类型放到一个容器中，java里Map就是最常用的容器之一

```java
@Component
public class InspectionSolverChooser implements ApplicationContextAware{


    private Map<String, InspectionSolver> chooseMap = new HashMap<>();

    public InspectionSolver choose(String type) {
        return chooseMap.get(type);
    }

    @PostConstruct
    public void register() {
        Map<String, InspectionSolver> solverMap = context.getBeansOfType(InspectionSolver.class);
        for (InspectionSolver solver : solverMap.values()) {
            for (String support : solver.supports()) {
                chooseMap.put(support,solver);
            }
        }
    }

    private ApplicationContext context;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.context=applicationContext;
    }
}
```

这里是在应用启动的时候，加载spring容器中所有InspectionSolver类型的处理器，放到InspectionSolverChooser的map容器中。注意是InspectionSolver类型，所以定义的处理器都得继承InspectionSolver，其次是spring容器中的才能加载，所以定义的处理器都得放到spring容器中（@Component注解不能少）

#### 3、定义不同的处理器

```java
@Component
public class ChangeWarehouseSolver extends InspectionSolver {

    @Override
    public void solve(Long orderId, Long userId) {
        System.out.println("订单"+orderId+"开始进行批量转仓了。。");
    }

    @Override
    public String[] supports() {
        return new String[] {InspectionConstant.INSPECTION_TASK_TYPE_BATCH_CHANGE_WAREHOUSE};
    }
}

@Component
public class ChangeShippingSolver extends InspectionSolver{

    @Override
    public void solve(Long orderId, Long userId) {
        System.out.println("订单"+orderId+"开始进行转快递了。。");
    }

    @Override
    public String[] supports() {
        return new String[] {InspectionConstant.INSPECTION_TASK_TYPE_BATCH_CHANGE_SHIPPING};
    }
}

@Component
public class ReplaceOrderGoodsSolver extends InspectionSolver{

    @Override
    public void solve(Long orderId, Long userId) {
        System.out.println("订单"+orderId+"开始进行替换商品了");
    }

    @Override
    public String[] supports() {
        return new String[]{InspectionConstant.INSPECTION_TASK_TYPE_BATCH_REPLACE_ORDER_GOODS};
    }
}
```

#### 4、测试类

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes=Application.class)// 指定spring-boot的启动类
public class InspectionTest {

    @Autowired
    private InspectionSolverChooser chooser;

    @Test
    public void test() throws Exception{
        //准备数据
        String taskType = InspectionConstant.INSPECTION_TASK_TYPE_BATCH_CHANGE_WAREHOUSE;
        Long orderId = 12345L;
        Long userId = 123L;
        //获取任务类型对应的solver
        InspectionSolver solver = chooser.choose(taskType);
        if (solver == null) {
            throw new RuntimeException("任务类型暂时无法处理!");
        }
        //调用不同solver的方法进行处理
        solver.solve(orderId,userId);
    }
}
```

在测试类中我消除了可能一长段的if else，从选择器InspectionSolverChooser中根据type的不同取出不同的任务处理器InspectionSolver，然后调用其solve()方法进行任务处理，不同处理器调用的当然就是不同的solve()方法了，目的达到