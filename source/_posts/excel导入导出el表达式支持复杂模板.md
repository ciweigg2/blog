cover: http://ciwei2.cn-sh2.ufileos.com/35.jpg
title: excel导入导出el表达式支持复杂模板
date: 2018-09-23 19:33:02
tags: [excel]
categories: [综合]
---
# poi-el
excel导出与导入神器，poi-el。  
* poi-el支持强大的excel模板导出功能  
* poi-el能很方便的支持导出函数扩展（EL的特性）  
* poi-el支持方便的excel导入API  

<!--more-->

## excel模板导出

使用poi + spEl，支持各种普通模板和复杂模板的导出功能

### Quick Start:

参考：  
com.kvn.poi.export_test.ForeachTest.java  
com.kvn.poi.export_test.MixTemplateTest.java  
com.kvn.poi.export_test.MultiPoiForeachTest.java  
  
## API: 

> PoiExporter.export2Destination(InputStream templateInputStream, Map<String, Object> rootObjectMap, OutputStream des)
	
## 模板示例  

## 普通模板：  

![](/images/excel1.png)

## 多sheet模板

![](/images/excel12.png)

## 复杂模板——多个foreach模板：  

![](/images/excel123.jpg)
  
## 复杂模板——混合模板：  

![](/images/excel混合模板.png)

## 导出函数扩展  
* 注册扩展函数到上下文中

```
poi-el默认会去读取 **META-INF/poi-el/poi-el-functions** 文件，里面存放扩展函数类的全限定名。  
poi-el内部默认注册了 [InternalUtils](src/main/java/com/kvn/poi/exp/function/InternalUtils.java)  
# 内部函数
com.kvn.poi.exp.function.InternalUtils
当然，开发者也可以将自定义的函数扩展添加到自己的工程目录文件 META-INF/poi-el/poi-el-functions 下
使用形如：（参考：src/test/resources/template/03_Template_混合模板.xlsx）
```

```
${#fmtDate(endDate,'yyyy-MM-dd')}
```

* 直接使用没有注册到上下文中的函数  
我们可以直接使用没有注册到上下文中的函数，不过需要指定方法的全限定名：（EL调用函数，是不是很强大^_^）  
例如：  

```
#{ T(com.kvn.poi.exp.function.NotRegistedFunction).fmtString(keyName) }
```
  
## excel模板导入

支持简洁易用的excel导入API  

### Quick Start:

参考:  
com.kvn.poi.import_test.ImportRawTest.java  
com.kvn.poi.import_test.ImportGenericTest.java  
  
## API
> PoiSheetVo sheetVo = PoiImporter.importFirstSheetFrom(is);

> PoiGenericSheetVo<OrderImportVo> genericSheetVo = PoiImporter.importFirstSheetFrom(is, OrderImportVo.class);
	
demo地址：https://gitee.com/ciweigg/poi-el