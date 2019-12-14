title: 还在重复写空指针检查代码？考虑使用 Optional 吧！
date: 2019-10-26 15:41:52
tags: [guava]
categories: [综合]
---
### 使用

> ifPresent

通常情况下，空指针检查之后，如果对象不为空，将会进行下一步处理，比如打印该对象。

```java
Company company = ...;
if(company!=null){
    System.out.println(company);
}
```

<!--more-->

上面代码我们可以使用 Optional#ifPresent 代替，如下所示：

```java
Optional<Company> optCompany = ...;
optCompany.ifPresent(System.out::println);
```

使用 ifPresent 方法，我们不用再显示的进行检查，如果 Optional 为空，上面例子将不再输出。

> filter

有时候我们需要某些属性满足一定条件，才进行下一步动作。这里假设我们当 Company name 属性为 Apple，打印输出 ok。

```java
if (company != null && "Apple".equals(company.getName())) {
    System.out.println("ok");
}
```

下面使用 Optional#filter 结合 Optional#ifPresent 重写上面的代码，如下所示：

```java
Optional<Company> companyOpt=...;
companyOpt
        .filter(company -> "Apple".equals(company.getName()))
        .ifPresent(company -> System.out.println("ok"));
```

filter 方法将会判断对象是否符合条件。如果不符合条件，将会返回一个空的 Optional 。

> orElseThrow

当一个对象为 null 时，业务上通常可以设置一个默认值，从而使流程继续下去。

```java
String name = company != null ? company.getName() : "Unknown";
```

或者抛出一个内部异常，记录失败原因，快速失败。

```java
if (company.getName() == null) {
    throw new RuntimeException();
}
```

Optional 类提供两个方法 orElse 与 orElseThrow ，可以方便完成上面转化。

```java
// 设置默认值
String name=companyOpt.orElse(new Company("Unknown")).getName();

// 抛出异常
String name=companyOpt.orElseThrow(RuntimeException::new).getName();
```

如果 Optional 为空，提供默认值或抛出异常。

> flatMap

熟悉 Java8 Stream 同学的应该了解，Stream#map 方法可以将当前对象转化为另外一个对象， Optional#map 方法也与之类似。

```java
Optional<Company> optCompany = ...;
Optional<String> nameopt = optCompany.map(Company::getName);
```

map 方法可以将原先 Optional 转变成 Optional ，此时 Optional 内部对象变成 String 类型。如果转化之前 Optional 对象为空，则什么也不会发生。

另外 Optional 还有一个 flatMap 方法，两者区别见下图。

![](/images/mapvsflatmap-2b57bab4.png)

`Department#getCompany 返回对象为 Optional`

### 代码重构

代码重构前：

```java
if (staff != null) {
    Department department = staff.getDepartment();
    if (department != null) {
        Company company = department.getCompany();
        if (company != null) {
            return company.getName();
        }
    }
}
return "Unknown";
```

首先我们需要将 Staff ，Department 修改 getter 方法返回结果类型改成 Optional。

```java
public class Staff {
    private Department department;
    public Optional<Department> getDepartment() {
        return Optional.ofNullable(department);
    }
    ...
}
public class Department {

    private Company company;
    public Optional<Company> getCompany() {
        return Optional.ofNullable(company);
    }
    ...
}

public class Company {
    private String name;
    public String getName() {
        return name;
    }
    ...
}
```

然后综合使用 Optional API 重构代码如下：

```java
Optional<Staff> staffOpt =...;
staffOpt
        .flatMap(Staff::getDepartment)
        .flatMap(Department::getCompany)
        .map(Company::getName)
        .orElse("Unknown");
```

可以看到重构之后代码利用 Optional 的 Fluent Interface，以及 lambda 表达式，使代码更加流畅连贯