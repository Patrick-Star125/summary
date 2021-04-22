## TensorFlow-v2.1.0

### tensorflow底层概念

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
- **秩**：张量的维度数量。标量的秩为 0，向量的秩为 1，矩阵的秩为 2。
- **轴**或**维度**：张量的一个特殊维度。
- **大小**：张量的总项数，即乘积形状向量

### Keras

one-hot处理

```python
iris_target = np.float32(tf.keras.utils.to_categorical(iris_target, num_classes=3))
```

输入层input

```python
input_xs  = tf.keras.Input(shape=(4), name='input_xs')
```

中间层Dens

```python
x = tf.keras.layers.Dense(32, activation='relu')(inputs)
```

模型保存与调用

```python
model.save()
new_model = tf.keras.models.load_model('./saver/the_save_model.h5')
model = tf.keras.Model(inputs=inputs, outputs=predictions)
```

compile函数Tensorflow2.0专用于配置训练模型的编译函数

fit函数用于对输入数据进行修改(泛化)

 [The Sequential model  | TensorFlow Core](https://www.tensorflow.org/guide/keras/sequential_model) 

[Module: tf.keras  | TensorFlow Core v2.4.0](https://www.tensorflow.org/api_docs/python/tf/keras) 

### Dataset



## Pytorch-v1.7