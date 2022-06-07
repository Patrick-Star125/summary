### CSS

- CSS 指层叠样式表 (**C**ascading **S**tyle **S**heets)
- 样式定义**如何显示** HTML 元素
- 样式通常存储在**样式表**中
- **外部样式表**可以极大提高工作效率
- 外部样式表通常存储在 **CSS 文件**中

**样式插入的三种方法**

**内联样式**

由于要将表现和内容混杂在一起，内联样式会损失掉样式表的许多优势。请慎用这种方法，例如当样式仅需要在一个元素上应用一次时。

要使用内联样式，你需要在相关的标签内使用样式（style）属性。Style 属性可以包含任何 CSS 属性。本例展示如何改变段落的颜色和左外边距。

```
<h1 style="color: brown;">xjl 永远的神</h1>
```

**内部样式表**

当单个文档需要特殊的样式时，就应该使用内部样式表。你可以使用 <style> 标签在文档头部定义内部样式表，就像这样:

```
<head>
	<title>Document</title>
	<style>
		a {color:yellow;}
	</style>
</head>
```

**外部样式表**

当样式需要应用于很多页面时，外部样式表将是理想的选择。在使用外部样式表的情况下，你可以通过改变一个文件来改变整个站点的外观。每个页面使用<link> 标签链接到样式表。 <link> 标签在（文档的）头部：

```
<head>
<link rel="stylesheet" type="text/css" href="mystyle.css">
</head>
```

**CSS语法**

CSS规则由两个主要的部分构成，分别是选择器和一条或多条声明，选择器在内联样式的情况下可以省略

选择器通常是需要改变样式的 HTML 元素。

每条声明由一个属性和一个值组成。

属性（property）是希望设置的样式属性（style attribute）。每个属性有一个值。属性和值被冒号分开。

~~~css
css参考样式集合
一. 字体属性：(font)

1. 大小 {font-size: x-large;}(特大) xx-small;(极小) 一般中文用不到，只要用数值就可以，单位：PX、PD
2. 样式 {font-style: oblique;}(偏斜体) italic;(斜体) normal;(正常)

3. 行高 {line-height: normal;}(正常) 单位：PX、PD、EM

4. 粗细 {font-weight: bold;}(粗体) lighter;(细体) normal;(正常)

5. 变体 {font-variant: small-caps;}(小型大写字母) normal;(正常)

6. 大小写 {text-transform: capitalize;}(首字母大写) uppercase;(大写) lowercase;(小写) none;(无)

7. 修饰 {text-decoration: underline;}(下划线) overline;(上划线) line-through;(删除线) blink;(闪烁)



二. 常用字体： (font-family)

"Courier New", Courier, monospace, "Times New Roman", Times, serif, Arial, Helvetica, sans-serif, Verdana


三. 背景属性： (background)

1. 色彩 {background-color: #FFFFFF;}
2. 图片 {background-image: none;}

3. 重复 {background-repeat: no-repeat;}repeat-x(水平);repeat-y(垂直)

4. 滚动 {background-attachment: fixed;}(固定) scroll;(滚动)

5. 位置 {background-position: left;}(水平) top(垂直);

简写方法 {background:#000 url(..) repeat fixed left top;} /*简写·这个在阅读代码中经常出现.


四. 区块属性： (Block)

1. 字间距 {letter-spacing: normal;} 数值 
2. 对齐 {text-align: justify;}(两端对齐) left;(左对齐) right;(右对齐) center;(居中)

3. 缩进 {text-indent: 数值px;}

4. 垂直对齐 {vertical-align: baseline;}(基线) sub;(下标) sup;(上标) top; text-top; middle; bottom; text-bottom;

5. 词间距word-spacing: normal; 数值

6. 空格white-space: pre;(保留) nowrap;(不换行)

7. 显示 {display:block;}(块) inline;(内嵌) list-item;(列表项) run-in;(追加部分) compact;(紧凑) marker;(标记) table; inline-table; table-raw-group; table-header-group; table-footer-group; table-raw; table-column-group; table-column; table-cell; table-caption;(表格标题) /*display 属性的了解很模糊*/


css

五. 方框属性： (Box)

1. width:; height:; float:; clear:both; margin:; padding:; 顺序：上右下左



六. 边框属性： (Border)

1. border-style: dotted;(点线) dashed;(虚线) solid; double;(双线) groove;(槽线) ridge;(脊状) inset;(凹陷) outset; border-width:; 边框宽度

border-color:#;

简写方法border：width style color; /*简写*/


七. 列表属性： (List-style)

1. 类型list-style-type: disc;(圆点) circle;(圆圈) square;(方块) decimal;(数字) lower-roman;(小罗码数字) upper-roman; lower-alpha; upper-alpha;

2. 位置list-style-position: outside;(外) inside;

3. 图像list-style-image: url(..);


八. 定位属性： (Position)

1.Position: absolute; relative; static;

visibility: inherit; visible; hidden;

overflow: visible; hidden; scroll; auto;

clip: rect(12px,auto,12px,auto) (裁切)


九. CSS文字属性：

1. color : #999999; /*文字颜色*/

2. font-family : 宋体,sans-serif; /*文字字体*/

3. font-size : 9pt; /*文字大小*/

4. font-style:itelic; /*文字斜体*/

5. font-variant:small-caps; /*小字体*/

6. letter-spacing : 1pt; /*字间距离*/

7. line-height : 200%; /*设置行高*/

8. font-weight:bold; /*文字粗体*/

9. vertical-align:sub; /*下标字*/

10. vertical-align:super; /*上标字*/

11. text-decoration:line-through; /*加删除线*/

12. text-decoration: overline; /*加顶线*/

13. text-decoration:underline; /*加下划线*/

14. text-decoration:none; /*删除链接下划线*/

15. text-transform : capitalize; /*首字大写*/

16. text-transform : uppercase; /*英文大写*/

17. text-transform : lowercase; /*英文小写*/

18. text-align:right; /*文字右对齐*/

19. text-align:left; /*文字左对齐*/

20. text-align:center; /*文字居中对齐*/

21. text-align:justify; /*文字分散对齐*/

vertical-align属性

22. vertical-align:top; /*垂直向上对齐*/

23. vertical-align:bottom; /*垂直向下对齐*/

24. vertical-align:middle; /*垂直居中对齐*/

25. vertical-align:text-top; /*文字垂直向上对齐*/

26. vertical-align:text-bottom; /*文字垂直向下对齐*/


十. CSS边框空白

1. padding-top:10px; /*上边框留空白*/

2. padding-right:10px; /*右边框留空白*/

3. padding-bottom:10px; /*下边框留空白*/

4. padding-left:10px; /*左边框留空白

~~~

