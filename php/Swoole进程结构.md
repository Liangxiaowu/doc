##### 结构图

![image-20200222101321774](/Users/xiaowu/Library/Application Support/typora-user-images/image-20200222101321774.png)

1、Master 主进程

2、Manger 管理进程

3、Worker 工作进程

4、Task 异步工作进程



##### 五大IO模型

Blocking IO				堵塞			用户一直等待

NoneBloking IO		非堵塞		一边做其他的 每个一段时间看一次 轮询

Multiplexing IO		多路复用	支持多个

Sigal driven IO			信号驱动		用户空间设置一个信号 内核处理会处理这个事情

Asynchronous IO		异步		 某节点去做某事情



