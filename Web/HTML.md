# html

### 基本信息

- HTML 指的是超文本标记语言: **H**yper **T**ext **M**arkup **L**anguage
- HTML 不是一种编程语言，而是一种**标记**语言
- 标记语言是一套**标记标签** (markup tag)
- HTML 使用标记标签来**描述**网页
- HTML 文档包含了HTML **标签**及**文本**内容
- HTML文档也叫做 **web 页面**

**HTML元素语法**

- HTML 元素以**开始标签**起始
- HTML 元素以**结束标签**终止
- **元素的内容**是开始标签与结束标签之间的内容
- 某些 HTML 元素具有**空内容（empty content）**
- 空元素**在开始标签中进行关闭**（以开始标签的结束而结束）
- 大多数 HTML 元素可拥有**属性**
- 大多数 HTML 元素可以嵌套（HTML 元素可以包含其他 HTML 元素）
- HTML 文档由相互嵌套的 HTML 元素构成
- HTML嵌套的特性让我们在使用CSS的时候可以对一块的元素进行调整

以上是HTML的一些基础概念，从中我们可以提取出一些关键的信息。首先，一个html包含最基本的三大部分

* 标签
* 内容
* 属性

标签是html的基本框架，内容包含在标签之间，属性附着在标签之上。

事实上，在简单的界面书写上，我们总会用到这些基本的东西，首先是最基本的框架

~~~html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    
</body>
</html>
~~~

上部分的代码是在vscode编辑器中输入！即可生成的基本html框架。其中包括很多部分，我们说最重要的几个标签

 ><html>|<head>|<body	这三个部分是最基本的标签

其中<html>标志着整个html文件的开始与结束，其中可以设置唯一属性lang，这表示html展示的语言是什么。

### head

head标签里里面可以设置html页面的全局属性

~~~html
<head>
    <title>网页标题</title> //用title标签设置网页在浏览器标签栏中显示的标题
    <meta>s
</head>
~~~

## body

body标签内包含html页面内所有可视的样式，其中可以设置的样式是最多的

~~~html
<body>
    
</body>
~~~

