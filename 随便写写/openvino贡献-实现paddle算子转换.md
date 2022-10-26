# 总览

在暑假中期开始参加了Paddle Hackathon活动（[活动链接](https://github.com/PaddlePaddle/Paddle/issues/43938#openvino)），并完成了ceil、sum、group_norm这三个算子从paddle框架到openvion引擎的转化，这里记录一下实现过程，帮助后面的同学更快的入门。

了解openvion框架的同学都知道openvino能够很好的压缩深度学习模型，使其体积、内存占用减少，推理速度提升。而实际我们在模型开发过程中一般使用的是诸如paddle这样的深度学习框架。

这样带来一个问题，即使模型推理的逻辑是相同的，在paddle框架下和openvino框架下模型推理的执行过程也会有相当大的差别，如何跨越这一差别呢？这就是我所贡献的代码所属模块的功能，将paddle模型推理涉及到的算子核心逻辑通过openvino的方式重新叙述。

关于具体做法，我将从我所贡献的三个算子实现过程来表述，非常巧的是，我所实现算子的顺序与它们的实现难易程度是有逐渐升高（ceil<sum<group_nrom）的对应关系的，所以各位同学选择自己需要的程度来学习即可。

先介绍基础步骤，首先可以将算子开发分为以下四步：

**第一步：搭建开发环境**

关于开发环境搭建，openvino的源码编译在[wiki](https://github.com/openvinotoolkit/openvino/wiki)中有详细的说明，也可以参照下面的过程，我的环境就是简单的带有C++环境的Ubuntu20.04，注意要用Intel CPU设备的机器即可。

```bash
$ git clone https://github.com/openvinotoolkit/openvino.git
$ cd openvino
$ git submodule update --init --recursive
$ chmod +x install_build_dependencies.sh
$./install_build_dependencies.sh
```

如果你在国内，尽量挂梯子，这样安装依赖能够一步解决，当然没有的话也能手动安装，问题不大。

**第二步：编译**

与其它C++开源项目的编译过程类似，不同的是在openvino中cmake的编译选项比较多。

~~~bash
$ export OPENVINO_BASEDIR=`pwd`
$ mkdir build
$ cd build
$ cmake \
-DCMAKE_BUILD_TYPE= Release -DCMAKE_INSTALL_PREFIX="${OPENVINO_BASEDIR}/openvino_dist" \
-DPYTHON_EXECUTABLE=$(which python3) \
-DENABLE_MYRIAD=OFF \
-DENABLE_VPU=OFF \
-DENABLE_PYTHON=ON \
-DNGRAPH_PYTHON_BUILD_ENABLE=ON \
-DENABLE_DEBUG_CAPS=ON \
-DENABLE_CPU_DEBUG_CAPS=ON  \
-DENABLE_TESTS=ON \
..
$ make -j$(nproc); make install
~~~

关于这一部分有一些我发现可能有的坑，但是这并不是我这篇文章的重点，所以简单提一下：

* cmake的编译参数很重要，上面的编译参数并不适用于所有设备，甚至可能仅适用于我做的这几个任务，可以查看[wiki](https://github.com/openvinotoolkit/openvino/wiki/CMakeOptionsForCustomCompilation)来了解编译参数的含义来进行调整，cmake过程如果有问题可能会导致非常难以排查的错误。
* make过程中会进行很多的检查，报错中断概率不小，要耐心调试，第一次编译成功后就会顺利很多。

第一次编译时间比较长，建议在性能比较好的机器上开发。

**第三步：进行算子逻辑、单元测试的编写**

为openvino的paddle端实现算子映射不仅要实现算子逻辑，还要实现配套的单元测试，它们随算子的逻辑复杂程度不同而各不相同，我将我所实现的ceil、sum、group_norm这三个算子贴在下面，希望能够给你的开发提供一些参考。

**第四步：运行单元测试**

编译好之后可以使用项目目录`bin/intel64/Release`中编译出的程序`paddle_tests`来进行单元测试，命令中`paddle_opname`换成自己命名的算子单测名称。

~~~bash
$ cd bin/intel64/Release
$ ./paddle_tests --gtest_filter=PaddleFuzzyOpTest/FrontEndFuzzyOpTest.testOpFuzzy/paddle_opname*
~~~

运行的结果是这样的

![image](https://user-images.githubusercontent.com/69072522/187865234-5bdc9b7c-d46f-4d48-8243-2a2f6c56bf9d.png)

这里的paddle_sum_{1,2,3,4}就是实现并注册的单元测试，如果编译过程不正确，那么你可能不会看到你所注册的算子打印出来。这可能是你的单测代码有问题，关于这一点也可以参考我的代码实现。

# ceil

该任务的实现pr链接：[【PaddlePaddle Hackathon 3】Add Paddle ceil operator by Patrick-Star125](https://github.com/openvinotoolkit/openvino/pull/12368)

在这个任务中，我要为OpenVINO实现Paddle 算子ceil 转换，ceil为**向上取整运算函数**，这个任务相对简单，是熟悉openvino框架非常好的材料，在我的pr中测试与注册的代码就不多说了，格式都是一样的，最核心的文件`ceil.cpp`代码如下，其实代码只有一句，逻辑就是调用openvino已经实现的算子[Ceiling](https://github.com/openvinotoolkit/openvino/blob/master/docs/ops/arithmetic/Ceiling_1.md)，将计算结果返回即可。

openvino算子库文档：[openvino/opset9.md at master](https://github.com/openvinotoolkit/openvino/blob/master/docs/ops/opset9.md)

~~~c++
NamedOutputs ceil(const NodeContext& node) {
    return node.default_single_output_mapping({std::make_shared<default_opset::Ceiling>(node.get_input("X"))}, {"Out"});
}
~~~

这种结构就是openvino的paddle端算子开发的基础结构，输入数据承载在`NodeContext& node`中，通过计算后由方法`node.default_single_output_mapping`返回并命名为`Out`，就算是完成了一次算子逻辑的编写。

你可能会好奇`node`究竟是怎样的原理，我的理解是：在openvino框架中，算子构成节点，按照推理逻辑组成图，数据以流的方式在图中处理，就像流水线加工一样，所以在这里，ceil节点的作用就是调用Ceiling算子，并返回计算结果。

如果你觉得这个解释太粗糙，细致的解释可以在[openvino文档](https://docs.openvino.ai/latest/documentation.html)中找到，实际上我认为理解原理对于贡献的帮助因人而异，有人在理解原理后根据经验能够推断出**可能有哪些方法能够帮助自己实现目标**，但像我一样缺乏经验的学生可能即使理解了原理也并不能由此产生代码的灵感，所以我选择了先动手，在实践中加深理解。

# sum

推荐在理解（或者实现）了上面算子之后看这一个算子的实现。

下一个任务是为OpenVINO实现Paddle算子sum转换，该OP用于对输入的**一或多个**Tensor或LoDTensor求和。

该任务的实现pr链接：[【PaddlePaddle Hackathon 3】Add Paddle sum operator by Patrick-Star125](https://github.com/openvinotoolkit/openvino/pull/12545)

在该任务实现的过程中我遇到了一些问题：

**任务算子不清楚**

首先是对标paddle算子不清楚，如果直接在[paddl官网文档](https://www.paddlepaddle.org.cn/documentation/docs/zh/2.4rc/guides/index_cn.html)搜索sum算子，只能找到一个与任务描述功能大不相同的sum算子，当时我发现这一点后并没有直接开始开发，而是想首先找与任务描述更类似的算子，但是一直没找到，索性直接与赛事组委会联系，隔天便收到邮件，原来paddle与openvino有对标的版本，当前openvino对标的是paddle 1.8版本算子，所以一开始就找错方向了

将搜索文档改为v1.8就可以找到对应的sum api调用路径，现在想想如果当时不声不响的做下去，应该会浪费不少时间。这里也建议看到这儿的小伙伴如果不太熟悉自己所用框架和openvino之间的对应关系，一定要提issue问清楚情况，才能提升效率。

**方法调用不清楚**

下面是我合入时候的代码

~~~c++
NamedOutputs sum(const NodeContext& node) {
    auto data = node.get_ng_inputs("X");
    auto sum = data[0].get_node_shared_ptr();
    for (int i = 1; i < data.size(); i++) {
        sum = std::make_shared<default_opset::Add>(sum, data[i]);
    }
    return node.default_single_output_mapping({sum}, {"Out"});
}
~~~

可以看到逻辑很简单，sum是结果，如果有多个输入就将sum循环自加，如果没有其本身就是最终结果，返回sum。

而我一开始的版本是这样的：

~~~c++
NamedOutputs sum(const NodeContext& node) {
    auto data = node.get_ng_inputs("X");
    const auto input_type = data[0].get_element_type();
    const auto shape = data[0].get_shape();
    auto sum = data[0];
    for (int i = 1; i < data.size()-1; i++) {
        // check the consistency of inputs
        PADDLE_OP_CHECK(node, input_type == data[i].get_element_type(), "The types of inputs are not consistent!");
        PADDLE_OP_CHECK(node, shape == data[i].get_shape(), "The shapes of inputs are not consistent!");
        sum = std::make_shared<default_opset::Add>(sum, data[i]);
    }
    return node.default_single_output_mapping({std::make_shared<default_opset::Add>(sum, data[data.size()])}, {"Out"});
}
~~~

可以看出来非常混乱，而且还有明显的逻辑bug。简单来说有两个原因：

* **sum的初始化**没有使用函数`get_node_shared_ptr()`，致使sum在后面的数据类型一直都是不正确的，导致返回时报错，而我又用**auto**来声明类型导致过程不清晰，无奈只能调用函数结果充当返回值（当时其实是试出来的，不然编译都通不过）。
* paddle在**调用sum算子的时候**就会进行类型和形状检查，这里的`PADDLE_OP_CHECK`完全是多余的，而这一点在make编译过程中是不会体现出来的

这两个问题如果在正常的c++开发中很简单就能发现，但是在openvino的开发模式中，运行时调试比较困难，并且方法之间调用关系不太清晰，IDE不能帮到我们太多，小问题也会造成难以排查的错误，这反映出两点：

* openvino的paddle端开发模式还有待改善
* 要对代码过程了解足够清晰，不然遇到问题时，排错成本更高
* 多参考其它算子的实现代码，尽量找到简化的实现方法

# group_norm

该任务的实现pr链接：[【PaddlePaddle Hackathon 3】Add Paddle group_norm operator by Patrick-Star125](https://github.com/openvinotoolkit/openvino/pull/12329)

在这个任务中，我需要为OpenVINO实现 Paddle 算子group_norm转换。 group_norm 组归一化是将channels分为很多组，对每组求均值和方差，然后对每组进行归一化。

这个任务相对前两个来讲难度增加了不少，因为代码比较多这里就不贴出来了，感兴趣的童鞋可以看源码，你会发现代码逻辑和group_norm公式的计算过程是一样的，当然实现这个任务所费的时间，参考的资料都是最多的，完成它的过程中我对openvino的理解有了不少的提升。

但是，但是，这个任务的**核心**和上面两个任务其实是一样的，复杂度提升主要体现在**代码逻辑复杂度增加**和**调用API数量增多**，但是**核心原理是一样的**，前两个任务的经验在这里发挥了非常大的作用，让我能够相对顺畅的完成这个任务。

相信能够理解（最好实践）前两个任务的同学也能顺利的理解这个任务的实现逻辑，至此，你就拥有了在openvino的paddle端开发相对复杂算子的能力。







