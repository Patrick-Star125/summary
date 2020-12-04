# **Python**数据分析

## Numpy

### 概念

1. ndarray是一个具有矢量算数运算和复杂广播(不同大小数组之间的运算)能力且节省空间的多维数组
2. 对整组数据进行运算的数学函数
3. 有线性代数、随机数生成和傅里叶变换功能
4. 提供一些简单易用的c语言程序接口
   * numpy中的dtype对象是与其它系统的交互的源泉
   * astpye可转换数组类型

![屏幕截图 2020-11-14 143032](D:\Coder\Github\学习周记\image\屏幕截图 2020-11-14 143032.jpg)

### 方法

![屏幕截图 2020-11-14 142453](D:\Coder\Github\学习周记\image\屏幕截图 2020-11-14 142453.jpg)

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

### 操作、属性

索引、切片

* 二维数组索引中有且仅有一个[]号时，只表示行索引
*  Python是从0的位置开始 
*  跟列表最重要的区别是，数组切片是原始数组的视图。这意味着数据不会被复制，**任何修改都会直接反映到源数组上** 
*  花式索引跟切片不一样，它总是将数据复 **制到新数组中** 
*  花式索引（Fancy indexing）是一个Numpy术语，它指的是利用**整数数组进行索引** 

## Pandas

### 概念

* pandas以numpy为基础，有些对矩阵的操作对DataFrame也有效

* 分清一个操作是视图操作还是复制操作很有用,有些操作对Series和DataFrame的影响无法逆反

* Series对象由一组各种Numpy数据类型和一组数据索引组成

![屏幕截图 2020-11-15 152303](D:\Coder\Github\学习周记\image\屏幕截图 2020-11-15 152303.jpg)

### 方法

我快看不懂中文的笔记了...

> data=pd.Series([数字、数组、字符串等等])
>
> data=pd.DataFrame(一个矩阵，行的索引、列的索引)
>
> 或者 data = pd.DataFrame(np.arange(12).reshape(3, 4))
>
> data.sort_index(axis=0或1,ascending=Ture or False)
>
> data.sort_values(by=index)
>
> data=pd.read_format-->data.to_format  # the operation to import and export is quite simple in pandas
>
> res = pd.concat([df1, df2, df3], axis=0, ignore_index=True)  **#** axis=0 means merging in vertical and axis=1 means in columns
>
> res = df6.append(df7, ignore_index=True)
>
> data.drop('name',)

### 操作、属性

> data.dtypes		data.index		data.cloums		data.values		data.T
>
> data.isnull()		data.notnull()		data.describe()		data2=data1.reindex
>
> data.sort_index(axis,ascending)		data.sor_values(by)		data.rank(method)

## Matpoltlib(data visualization)

### Matpolitib概述



### 基于numpy画图

### 基于pandas和 seaborn画图