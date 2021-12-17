### JavaScript

* JavaScript 是一种轻量级的编程语言。
* JavaScript 是可插入 HTML 页面的编程代码。
* JavaScript 插入 HTML 页面后，可由所有的现代浏览器执行。

**JS的用法**

HTML 中的脚本必须位于 <script> 与 </script> 标签之间。

脚本可被放置在 HTML 页面的 <body> 和<head> 部分中。


如需在 HTML 页面中插入 JavaScript，请使用 <script> 标签。

<script> 和 </script> 会告诉 JavaScript 在何处开始和结束。

比如：

```
<script>
alert("我的第一个 JavaScript");
</script>
```

你们暂时无需理解上面的代码。只需明白，浏览器会解释并执行位于 `<script> 和 </script>`之间的 JavaScript 代码 。

**写JS代码的三个位置**

1. 写在head当中
2. 写在body当中
3. 写在外置文件当中

```
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<meta http-equiv="X-UA-Compatible" content="IE=edge">
	<meta name="viewport" content="width=device-width, initial-scale=1.0">
  <!-- 写在外置文件当中 -->
  <script src="myScript.js"></script>
	<title>Document</title>
  <!-- 写在head中 -->
  <script>
		document.write("<h1>223333</h1>");
	</script>
</head>
<body>
  <!-- 写在body中 -->1
	<script>
		document.write("<h1>1111</h1>");
		alert("666，我的宝贝");
	</script>
</body>
</html>
```

写在body和写在head中的代码其实没有什么特殊的区别，需要注意的就是head中的JS代码是先被执行的，body中的代码是后被执行的。