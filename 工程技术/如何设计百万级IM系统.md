原文：[从0到1再到N，探索亿级流量的IM架构演绎 - 飞书云文档 (feishu.cn)](https://nxwz51a5wp.feishu.cn/docs/doccnTYWSZg4v9bYTQH8hXkGJPc)

# 功能和约束

~~~
1. 添加好友
2. 聊天会话列表
3. 单聊  用户A给用户B发消息
4. 群聊 多个用户在一个聊天室内聊天
5. 多设备登陆
6. 消息漫游
7. 消息已读，查看已读/未读列表
~~~

~~~
1. DAU 10亿 预估QPS和存储消耗
2. 可靠性要求5个9
3. 收发消息延迟在10ms以下
4. 消息时序一致性(发送与接收端的消息顺序一致，不重不漏)
5. 万人群聊
6. 可运维性
~~~

# 低延时

![image-20240308112710610](C:\Users\NetPunk\AppData\Roaming\Typora\typora-user-images\image-20240308112710610.png)