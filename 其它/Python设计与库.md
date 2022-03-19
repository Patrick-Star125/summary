# Python基础学习

**python特性**

*  而Python 属于脚本语言，不像编译型语言那样先将程序编译成二进制再运行，而是动态的逐行解释运行。也就是从脚本第一行开始运行，没有统一的入口，所以可以直接在文本编辑工具编写python。 
* 语法单薄，解决问题才是关键
* python语言为**动态类型语言**，就是多态的体现

## Python基本知识

**值、类型、对象 、函数、模块、运算**

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

**列表和元组**

* 容器有列表、元组、字典、集合
* python中没有专门用于表达字符的类型
* 负数索引就是反着索引，隐式步长与显示步长
* 列表和字符串不能相加拼接
* 可以用in来判别序列中是否有这个元素
* None、空列表可以强行拉长序列长度
* 切片是一项极其强大的功能，给切片赋值是其核心功能

**字符串**

* 设置字符串的格式用format,%和{}
* 格式设置可以对齐，指定字段宽度和精度，添加正负号和填充
* 字符串方法有一万个
* is打头的一堆函数用于判断字符串是否满足特定条件
* python中单引号和双引号在定义上没有区别，只有在提升程序可读性的时候需要注意其区别

**字典**

* 比起列表的索引，键是字典中最为关键的概念
* 字典视图是一个特殊类型，用于迭代。它不复制，一直底层字典的反映。
* 浅复制不影响对象，而深复制不同
* 字典是无序的，所有顺序相关处理均无效
* 对字典指向字符串格式设置操作，要用format_map

**条件、循环和其它语句**

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

**抽象与函数**

* 函数内容大致与c语言一致，但是更加自由，可以为函数编写说明
* 没有返回值的函数返回None
* 最好不要将两个变量指向同一个对象，应该创建副本
* 对于不可修改的参数，最好在函数外赋值。
* 关键字参数指定默认，可以灵活赋值
* 值与变量的关系就是字典，函数外变量可以用于函数内(有bug)
* 全局变量(global)可进，局部变量不可出(locals)
* 储存一个函数所在作用域的函数叫闭包
* 在python函数中带有一个或两个的星号，意味着可以将任意个数的参数导入到python中[教程](https://www.runoob.com/w3cnote/python-one-and-two-star.html)

**类和对象**

* 对象：一系列数据和一套访问和操作这些数据的方法。对象的特征：多态，封装，继承。
* 多态：可以无视对象是什么类的，就可调用其方法
* 多重继承可能存在方法继承问题
* 如果一个函数操作一个全局变量，最好将它们作为一个类的属性和方法
* 对象中(关联的)方法总是将所属的对象作为第一个参数(self)
* 多态是指能够同样的对待不同类型和类的对象
* 接口和内省：一些函数用于安全的查看对象拥有的属性

## Python基础进阶

**异常**

* 异常对象未被处理或捕获的话，则会终止程序

* 异常是一个个对象，可以主动引发异常，或者可以用警告某种程度代替异常

* 在可能出错的地方打上警告或异常

* try：有 try/except/else 风格和 try/finally 风格,包在try语句里的异常才能被特殊处理

* 捕获异常后打印异常位置（异常原因）

  ```python
  # encoding=utf-8
  import sys
  def get_cur_info():
      print sys._getframe().f_code.co_filename # 当前文件名，可以通过__file__获得
      print sys._getframe().f_code.co_name # 当前函数名
      print sys._getframe().f_lineno # 当前行号
  ```

**文件**

> f.open()		f.write()		f.read()		f.close()

* 文件模式默认是读取模式，写入模式从头开始写入，并创建不存在的文件，附加模式在文件末尾开始写
* 默认模式为'rt'，即将文件视为unicode文本，但一般用'rb'，即处理声音或图像等二进制文件
* 可以按行读取或写入，自行添加换行符
* 执行写入后一定要在文件处理完成后关闭文件(用with语句自动关闭文件)
* 类似于文件的对象、打开和关闭文件、模式和文件类型、标准流、迭代文件内容

**语法糖**

* python中，for...[if]...语句一种简洁的构建List的方法，从for给定的List中选择出满足if条件的元素组成新的List，其中if是可以省略的。返回一个满足if条件的list。

* 在函数参数中*parameter是用来表示接收任意多个参数并将其放在一个**元组**中，而对非函数参数，则是对可迭代对象进行解包，例如

  * ~~~python
    a=*range(3) 或者 b=[*range(3)]
    ~~~

* 

**装饰器**

* @property：我们可以使用@property装饰器来创建**只读属性**，@property装饰器会将**方法**转换为相同名称的**只读属性**,可以与所定义的属性配合使用，这样可以防止属性被修改。有两种主要用法。[python @property的介绍与使用 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/64487092)

## Python实用方法

**random**

~~~python
random() 返回0<=n<1之间的随机实数n；
choice(seq) 从序列seq中返回随机的元素；
getrandbits(n) 以长整型形式返回n个随机位；
shuffle(seq[, random]) 原地指定seq序列；
sample(seq, n) 从序列seq中选择n个随机且独立的元素；
~~~





# Python数据科学库

## Numpy

**概念**

1. ndarray是一个具有矢量算数运算和复杂广播(不同大小数组之间的运算)能力且节省空间的多维数组
2. 对整组数据进行运算的数学函数
3. 有线性代数、随机数生成和傅里叶变换功能
4. 提供一些简单易用的c语言程序接口
   * numpy中的dtype对象是与其它系统的交互的源泉
   * astpye可转换数组类型

<img src="http://1.14.100.228:8002/images/2022/01/11/-2020-11-14-143032.png" style="zoom:50%;" />

**方法**

![](http://1.14.100.228:8002/images/2022/01/11/-2020-11-14-142453.png)

> data=np.array([1,2,3])
>
> data=np.array([[[1,2],[2,3]],[[3,4],[4,5]]])     //二维数组
>
> data=np.asarray()    # 列表转换为ndarray对象
>
> data=np.ones(3)
>
> data=np.zeros(3)
>
> data=np.random.random(3)
>
> data=np.max(axis=0) # 按列聚合    data=np.min(axis=1)  # 按行聚合
>
> data=np.sum()
>
> data[n,m]  # 数组索引
>
> data1.dot(data2)  # 矩阵点积
>
> data.T  矩阵转置
>
> data.reshape  # 矩阵重置(改变矩阵形状)
>
> data.detype # 推断数组存放数据类型

**使用技巧**

索引、切片

* 二维数组索引中有且仅有一个[]号时，只表示行索引
* Python是从0的位置开始 
* 跟列表最重要的区别是，数组切片是原始数组的视图。这意味着数据不会被复制，**任何修改都会直接反映到源数组上** 
* 花式索引跟切片不一样，它总是将数据复 **制到新数组中** 
* 花式索引（Fancy indexing）是一个Numpy术语，它指的是利用**整数数组进行索引** 

## Pandas

**概念**

* pandas以numpy为基础，有些对矩阵的操作对DataFrame也有效

* 分清一个操作是视图操作还是复制操作很有用,有些操作对Series和DataFrame的影响无法逆反

* Series对象由一组各种Numpy数据类型和一组数据索引组成

![](http://1.14.100.228:8002/images/2022/01/11/-2020-11-15-152303.png)

**方法**

pandas查看数据的基本信息

~~~python
df.info():          # 打印摘要
df.describe():      # 描述性统计信息
df.values:          # 数据 <ndarray>
df.to_numpy()       # 数据 <ndarray> (推荐)
df.shape:           # 形状 (行数, 列数)
df.columns:         # 列标签 <Index>
df.columns.values:  # 列标签 <ndarray>
df.index:           # 行标签 <Index>
df.index.values:    # 行标签 <ndarray>
df.head(n):         # 前n行
df.tail(n):         # 尾n行
pd.options.display.max_columns=n: # 最多显示n列
pd.options.display.max_rows=n:    # 最多显示n行
df.memory_usage():                # 占用内存(字节B)
~~~



> data=pd.Series([数字、数组、字符串等等])
>
> data=pd.DataFrame(一个矩阵，行的索引、列的索引)
>
> data.sort_index(axis=0或1,ascending=Ture or False)
>
> data.sort_values(by=index)
>
> data=pd.read_format-->data.to_format  # the operation to import and export is quite simple in pandas
>
> res = pd.concat([df1, df2, df3], axis=0, ignore_index=True)  
>
> res = df6.append(df7, ignore_index=True)
>
> data.drop('name',)

**使用技巧**

## Scikit-learn

先积累代码里的import看看吧

```python
from sklearn.metrics import roc_curve, auc, confusion_matrix
import  sklearn.preprocessing.OneHotEncoder
from sklearn.model_selection import StratifiedKFold
from sklearn.preprocessing import StandardScaler, MinMaxScaler
from sklearn.model_selection import KFold, cross_val_score, train_test_split, GridSearchCV
from sklearn.linear_model import LinearRegression
from sklearn.metrics import mean_squared_error, r2_score
from sklearn.decomposition import PCA
from sklearn.linear_model import ElasticNet, Lasso, BayesianRidge, LassoLarsIC, LinearRegression
from sklearn.svm import SVR
from sklearn.ensemble import RandomForestRegressor, GradientBoostingRegressor
from sklearn.kernel_ridge import KernelRidge
from sklearn.pipeline import make_pipeline
from sklearn.preprocessing import RobustScaler
from sklearn.base import BaseEstimator, TransformerMixin, RegressorMixin, clone
from sklearn.feature_selection import VarianceThreshold
from sklearn.feature_selection import SelectKBest
from sklearn.feature_selection import f_regression
from sklearn.feature_selection import SequentialFeatureSelector
```

具体的sklearn的逻辑还没有梳理出来，API实在是太多了















