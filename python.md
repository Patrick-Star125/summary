# Python基础学习

### python特点

*  而Python 属于脚本语言，不像编译型语言那样先将程序编译成二进制再运行，而是动态的逐行解释运行。也就是从脚本第一行开始运行，没有统一的入口，所以可以直接在文本编辑工具编写python。 
* 语法单薄，解决问题才是关键
* python语言为**动态类型语言**，就是多态的体现

## Python基本概念

### 1.值、类型、对象 、函数、模块、运算

* 基本运算符和其可运算的对象
* 所有的数据都以对象的形式存在，对象包含值、数据类型、和id
* python是动态类型语言，只有运行时根据变量引用的对象才能知道这个对象的数据类型
* int可以任意长，浮点型精度15位小数
* **列表**：相当自由的数组，下标范围只能用list对象赋值
* 如果字符串内容写在多行，则要用三个引号，单(双)引号里不能直接单(双)引号
* 转义字符算一个字符
* 集合根据元素[哈希值](什么玩意儿？)存储元素
* 显示类型转换，int,str,float
* 可以用变量来引用函数和其它元素
* python没有表示虚数的概念，把虚数视为实部为零的复数

### 2.列表和元组

* 容器有列表、元组、字典、集合
* python中没有专门用于表达字符的类型
* 负数索引就是反着索引，隐式步长与显示步长
* 列表和字符串不能相加拼接
* 可以用in来判别序列中是否有这个元素
* None、空列表可以强行拉长序列长度
* 切片是一项极其强大的功能，给切片赋值是其核心功能

### 3.字符串

* 设置字符串的格式用format,%和{}
* 格式设置可以对齐，指定字段宽度和精度，添加正负号和填充
* 字符串方法有一万个
* is打头的一堆函数用于判断字符串是否满足特定条件

### 4.字典

* 比起列表的索引，键是字典中最为关键的概念
* 字典视图是一个特殊类型，用于迭代。它不复制，一直底层字典的反映。
* 浅复制不影响对象，而深复制不同
* 字典是无序的，所有顺序相关处理均无效
* 对字典指向字符串格式设置操作，要用format_map

### 5.条件、循环和其它语句

* print要么打印字符串，要么把打印的转换为字符串
* 模块和模块内函数都可以重新命名
* 用=序列解包，用*回收多余元素返回序列
* 布尔运算符也有短路逻辑
* for循环依然更牛逼，用于遍历、解包、[列表推导](完全看不懂)
* i是用做循环索引的变量的标准名称
* 循环体庞大复杂，存在多个要跳过的原因
* 这个else语句有点东西
* 删除变量用del，而值是无法通过手动删除的
* 作用域、命名空间以及代码实现(好难啊)
* 推导并不是语句，而是表达式，可从既有列表创建出新列表，也可创建字典

### 6.抽象与函数

* 函数内容大致与c语言一致，但是更加自由，可以为函数编写说明
* 没有返回值的函数返回None
* 最好不要将两个变量指向同一个对象，应该创建副本
* 对于不可修改的参数，最好在函数外赋值。
* 关键字参数指定默认，可以灵活赋值
* 值与变量的关系就是字典，函数外变量可以用于函数内(有bug)
* 全局变量(global)可进，局部变量不可出(locals)
* 储存一个函数所在作用域的函数叫闭包
* 我佛了，东西越来越多了

### 7.类和对象

* 对象：一系列数据和一套访问和操作这些数据的方法。对象的特征：多态，封装，继承。
* 多态：可以无视对象是什么类的，就可调用其方法
* 多重继承可能存在方法继承问题
* 如果一个函数操作一个全局变量，最好将它们作为一个类的属性和方法
* 对象中(关联的)方法总是将所属的对象作为第一个参数(self)
* 多态是指能够同样的对待不同类型和类的对象
* 接口和内省：一些函数用于安全的查看对象拥有的属性

## Python进阶

##### 异常

* 异常对象未被处理或捕获的话，则会终止程序
* 异常是一个个对象，可以主动引发异常，或者可以用警告某种程度代替异常
* 在可能出错的地方打上警告或异常
* try：有 try/except/else 风格和 try/finally 风格,包在try语句里的异常才能被特殊处理

### 文件

> f.open()		f.write()		f.read()		f.close()

* 文件模式默认是读取模式，写入模式从头开始写入，并创建不存在的文件，附加模式在文件末尾开始写
* 默认模式为'rt'，即将文件视为unicode文本，但一般用'rb'，即处理声音或图像等二进制文件
* 可以按行读取或写入，自行添加换行符
* 执行写入后一定要在文件处理完成后关闭文件(用with语句自动关闭文件)
* 类似于文件的对象、打开和关闭文件、模式和文件类型、标准流、迭代文件内容

