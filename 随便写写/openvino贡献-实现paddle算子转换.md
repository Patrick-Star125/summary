# 总览

在暑假中期开始参加了Paddle Hackathon活动（[活动链接](https://github.com/PaddlePaddle/Paddle/issues/43938#openvino)），并完成了ceil、sum、group_norm这三个算子从paddle框架到openvion引擎的转化，了解openvion框架的同学都知道openvino能够很好的压缩深度学习模型，使其体积、内存占用减少，推理速度提升。而实际我们在模型开发过程中一般使用的是诸如paddle这样的深度学习框架。

这样带来一个问题，即使模型推理的逻辑是相同的，在paddle框架下和openvino框架下模型推理的执行过程也会有相当大的差别，如何跨越这一差别呢？这就是我所贡献的代码所属模块的功能，将paddle模型推理涉及到算子的核心逻辑通过openvino的方式重新叙述。

具体做法我从我所贡献的三个算子实现过程来表述，非常巧的是，我所实现算子的顺序与它们的实现难易程度是有逐渐升高（ceil<sum<group_nrom）的对应关系的，所以各位看官选择自己需要的程度来学习即可。

# ceil

该任务的实现pr链接：[【PaddlePaddle Hackathon 3】Add Paddle ceil operator by Patrick-Star125](https://github.com/openvinotoolkit/openvino/pull/12368)

在这个任务中，我要为 OpenVINO 实现 Paddle 算子 ceil 转换，ceil为**向上取整运算函数**，这个任务相对简单，对

# sum



# group_norm