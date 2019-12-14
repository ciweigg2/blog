title: IDEA插件
date: 2018-11-02 17:36:59
tags: [idea]
categories: [IDEA插件]
---
## IDEA常用工具

<!--more-->

### CamelCase

使用方法:

将不是驼峰格式的名称，快速转成驼峰格式，安装好后，选中要修改的名称，按快捷键shift+alt+u。

### Material Theme UI

这是一款主题插件，可以让你的ide的图标变漂亮，配色搭配的很到位，还可以切换不同的颜色，甚至可以自定义颜色。默认的配色就很漂亮了，如果需要修改配色，可以在工具栏中Tools->Material Theme然后修改配色等。

### POJO to JSON

选中类-右键-MakeJson 将简单Java类型转成JSON

 方便用postman或者curl的时候构造JSON body

![](/images/screenshot_16960.png)

CTRL+V查看转换的JSON

### json2pojo with Lombok

IntelliJ Idea插件，从JSON文本生成POJO, 并添加Lombok与Gson/Jackson注解.

* 安装

从plugin库marketplace搜索`json2pojo with Lombok`。

* 使用

右键目标package，选择"New-> Convert JSON to POJOs"

* Example

运行 `GeneratorTest`，生成的主类:
```java
    package example.spark;

    import java.util.List;
    import com.google.gson.annotations.SerializedName;
    import lombok.Data;
    import lombok.RequiredArgsConstructor;

    @RequiredArgsConstructor
    @Data
    @SuppressWarnings("unused")
    public class SparkProgress {
        @SerializedName("batch.id")
        private long batccId;
        private DurationMs durationMs;
        private String id;
        @SerializedName("input-rows-per-second")
        private double inputRowsPerSecond;
        private String name;
        @SerializedName("num_input_rows")
        private long numInputRows;
        private double processedRowsPerSecond;
        private String runId;
        private Sink sink;
        private List<Source> sources;
        private List<StateOperator> stateOperators;
        private String timestamp;
    }
```

### PojoToJson

将实体类转成json格式支持yapi dubbo type 类型的转换

idea中搜索PojoToJson

右击实体类

选择需要生成的类型

![](/images/20190211171255.png)

### SONAR性能分析插件

* IDEA中搜索SonarLint

### Git Commit Template

* IDEA提交模板

### Gitmoji

![](/images/screenshot_17718.png)

### StringManipulation

https://plugins.jetbrains.com/plugin/2162-string-manipulation

可以将String字符串格式化成驼峰排序字符串等

### restfultoolkit

Java WEB开发必备，再也不用全局搜索RequestMapping了

![](/images/201908161050123.jpg)

![](/images/20190816105358.png)

### Yet another emoji support

This plugin supports Go, Groovy, Java, JavaScript, Kotlin, Markdown, PHP, Python, Ruby, Rust, Scala, TypeScript

![](/images/screenshot_19756.gif)

![](/images/screenshot_19757.png)

### JBLSpringBootAppGen

JBLSpringBootAppGen 简介

在使用SpringBoot项目的时候都需要创建启动引导类**Application； 使用该插件可以快速创建启动引导类**Application类内容

在IDEA模块工程上右击点击“JBLSpringBootAppGen”按照填写的全限定类名；直接生成**Application启动引导类

### get emoji

代码中添加表情

快捷键 command+shift+i

![](/images/WX20191201-133924@2x.png)

### extra icons

更换图标插件

![](/images/screenshot_18524.png)

### javadoc注释自动生成

支持自定义模板

常用的方法注释模板(yapi生成文档的模板)

```bash
/**
 * $DOC$
 * @author $AUTHOR$
 * @date $date$
 * @description $DOC$
 * $params$
 * @status 已发布
 * @return $return$
 */
```

可以配合smart-doc生成文档 也可以平时添加注解的时候使用

打开IntelliJ IDEA -> plugin，搜索 easy-javadoc，安装重启即可

打开配置页面

![](/images/WX20191211-103223@2x.png)

自定义单词配置呀

![](/images/20190901155929.jpg)

源码地址：https://github.com/starcwang/easy_javadoc

![](/images/k03vffH6Hg.gif)

支持给中文起名字，类似程序员起名神器

![](/images/zqT2bjDzc0.gif)

将光标放置到想要生成注释的类、方法或者属性上，然后按下快捷键ctrl \或者command \，即可生成注释，你的方法名起的越贴切，注释越得体。
将光标放置到想要生成注释的类上，然后按下快捷键ctrl shift \或者command shift \，即可批量生成文档注释