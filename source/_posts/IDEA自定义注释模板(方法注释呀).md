title: IDEA自定义注释模板(方法注释呀)
date: 2019-08-28 21:55:26
tags: [idea]
categories: [综合]
---
### 介绍

主要规范代码注释 生成yapi文档需要 按照规范编码接口注释可以方便生成yapi文档

<!--more-->

### 添加模板

settings -> Editor -> Live Templates

新建自己的分组和自己的模板

![](/images/20190828215843.png)

Abberviation填写*

重点：Abbreviation那里不要用/开头的

重点：模板中开头不要/，从*号开始 模板如下：

Template text填写

```
*
 * @author $user$
 * @date $date$ $time$
 * @description 
 $param$
 * @return {@link $return$}
 **/
```

类型选择java

![](/images/20190828220131.png)

其中params变量的内容一定要放在Default value中：

```
groovyScript("if(\"${_1}\".length() == 2) {return '';} else {def result=''; def params=\"${_1}\".replaceAll('[\\\\[|\\\\]|\\\\s]', '').split(',').toList();for(i = 0; i < params.size(); i++) {if(i==0){result+='* @param ' + params[i] +' '}else{result+='\\n' + ' * @param ' + params[i] + ' '}}; return result;}", methodParameters());
```

注释时需要自己打/符号，然后再打*，然后tab，这样就可以获取了

/** 然后tab 就能输出模板了

### 结合yapi插件使用

idea中安装RedsoftYapiUpload 通过上面的方法生成注释 然后选中类或者方法右击上传呀

具体参考：https://github.com/diwand/YapiIdeaUploadPlugin

也可以使用idea插件easy_javadoc生成自定义模板