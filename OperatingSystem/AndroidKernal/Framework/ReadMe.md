第1章 系统服务相关面试问题

本章重点讲解系统核心进程，以及一些关键的系统服务的启动原理和工作原理相关的面试内容。
1 谈谈对zygote的理解
    Zygote的作用是什么？
        对于Zygote的作用实际上可以概括为以下两点：
    * 创建SystemServer
    * 孵化应用进程
    https://zhuanlan.zhihu.com/p/260414370
 

2 说说Android系统的启动
3 你知道怎么添加一个系统服务吗？
    https://blog.csdn.net/menghaocheng/article/details/104316165
4 系统服务和bind的应用服务有什么区别？
    https://www.jianshu.com/p/f633fda5acc6
5 ServiceManager启动和工作原理是怎样的？
6 谈谈对AMS的理解
第2章 应用进程相关面试问题

本章主要讲解应用进程的启动，以及伴随进程启动过程中的一些重要机制的初始化原理，比如binder机制，Application，以及Context等方面的面试问题。
1 你知道应用进程是怎么启动的吗？

2 应用是怎么启用Binder机制的？

3 谈谈你对Application的理解

4 谈谈你对Context的理解
5 冷启动App
第3章 Activity组件相关面试问题

这一章主要讲解Activity相关的机制，包括Activity的启动流程，显示原理等相关面试问题，通过本章的学习，我们不但能熟悉它，更能深入了解它。
1 说说Activity的启动流程
2 说说Activity的显示原理
3 应用的UI线程是怎么启动的
第4章 其它应用组件相关面试问题

本章主要讲除了Activity之外的应用组件相关面试问题，包括service的启动和绑定原理，静态广播和动态广播的注册和收发原理，provider的启动和数据传输原理等等。
1 说说service的启动原理

2 说说service的绑定原理-1

3 说说service的绑定原理-2

4 说说动态广播的注册和收发原理

5 说说静态广播的注册和收发原理

6 说说Provider的启动原理
第5章 UI体系相关面试问题

本章主要讲UI体系相关面试问题，包括UI刷新机制，涉及到vsync和choreographer原理。另外还会讲到surface的相关原理，涉及到应用和WMS、surfaceFlinger通信。
1 说说屏幕刷新的机制-1

2 说说屏幕刷新的机制-2

3 surface跨进程传递原理

4 surface的绘制原理

5 你对vsync机制有了解吗？

6 SurfaceView & View的区别，底层原理有何不同
第6章 进程通信相关面试问题

本章主要讲进程通信相关面试问题，包括binder的整体架构和通信原理，oneway机制，binder对象的传递等等。
1 Android Framework用到了哪些跨进程通信方式

2 谈谈你对Binder的理解

3 一次完整的ipc通信流程是怎样的

4 binder对象跨进程传递原理是怎么样的

5 说一说binder的oneway机制
第7章 线程通信相关面试问题

本章主要讲线程通信原理相关面试问题，包括消息队列的创建，消息循环机制，消息延时，同步和异步消息，消息屏障等等内容。
1 线程的消息队列是怎么创建的？

2 说说android线程间消息传递机制

3 handler的消息延时是怎么实现的？

4 说说idleHandler的原理

5 主线程进入loop循环了为什么没有ANR？

6 听说过消息屏障么？

7 多线程间通信和多进程之间通信有什么不同，分别怎么实现？
第8章 技巧，心得相关

除了上面章节之外的所有问题，都会放在本章讲到，除了原理之外，还会分享一些代码技巧。
1 怎么跨进程传递大图片

2 说说threadLocal的原理

3 来说说looper的副业

4 怎么检查线程有耗时任务

5 怎么同步处理消息
6 主线程&ApplicationThread
第9章 综合性面试问题

本章主要是讨论一些综合性较强的面试题，这类题目不会问到具体某一块原理，需要充分结合自己的思考和积累，没有唯一的答案。本章我们就来讨论这些开放的题目该怎么答，有哪些思路可以借鉴的。
1 你去了解framework是为了解决一个什么样的问题，怎么解决的
2 Android Framework用到了哪些设计模式
3 Framework中有什么你觉得设计的很巧妙的地方，请举例说明-1
4 Framework中有什么你觉得设计的很巧妙的地方，请举例说明-2
