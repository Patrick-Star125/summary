## TensorFlow-v2.1.0

### 计算

##### 图(graph)

 一个`Graph`包含一系列`tf.Operation`对象和`tf.Tensor`对象；`tf.Operation`对象代表计算单元，`tf.Tensor`对象代表operation之间流动的数据单元). 

用 tf.Graph.as_default 创建图。

##### 会话(session)

 `Session`对象封装了执行`Operation`对象和计算`Tensor`对象的环境。  调用`tf.Session.close`方法 ，释放会话占用的资源。

 如果在构造会话时没有指定`graph`参数，则将在会话中启动默认图。但是每个图都可以在多个会话中使用。 

##### 变量(varible)

 [变量简介  | TensorFlow Core](https://www.tensorflow.org/guide/variable) 

 变量通过 `tf.Variable`类进行创建和跟踪。`tf.Variable`表示张量，对它执行运算可以改变其值。利用特定运算可以读取和修改此张量的值。大部分张量运算在变量上也可以按预期运行，不过变量无法重构形状。 

##### 张量(tensor)

 [张量简介  | TensorFlow Core](https://www.tensorflow.org/guide/tensor) 

 张量是具有统一类型（称为 `dtype`）的多维数组，与 `np.arrays` 有一定的相似性，就像 Python 数值和字符串一样，所有张量都是不可变的：永远无法更新张量的内容，只能创建新的张量。 

张量有形状。下面是几个相关术语：

- **形状**：张量的每个维度的长度（元素数量）。
- **秩**：张量的维度数量。标量的秩为 0，向量的秩为 1，矩阵的秩为 2
- **轴**或**维度**：张量的一个特殊维度。
- **大小**：张量的总项数，即乘积形状向量

### 模型

## Pytorch-v1.7

### 计算

### 模型

1. 模型保存为Orderdict可以使用生成器语法来进行来增删查改模型的某个层

### 特性