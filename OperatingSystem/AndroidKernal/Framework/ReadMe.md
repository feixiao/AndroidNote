## Andorid Framework入门
#### 第1章 系统服务相关面试问题
本章重点讲解系统核心进程，以及一些关键的系统服务的启动原理和工作原理相关的面试内容。
##### 1：谈谈对zygote的理解
Zygote的作用是什么？
对于Zygote的作用实际上可以概括为以下两点：
+ 创建SystemServer
+ 孵化应用进程

![](./imgs/zygote.jpeg) 

+ 参考[谈谈对Android中Zygote的理解](https://zhuanlan.zhihu.com/p/260414370)
 

##### 2: 说说Android系统的启动
Android是基于Linux系统的。但是它没有BIOS程序，取而代之的是**BootLoader（系统启动加载器）**。类似于BIOS，在系统加载前，用于初始化硬件设备，最终调用系统内核准备好环境。在Android中没有硬盘，而是ROM，类似于硬盘存放操作系统，用户程序等。

ROM跟硬盘一样也会划分为不同的区域，用于放置不同的程序，在Android中主要划分为以下几个区域：
+ /boot: 存放引导程序，包括内核和内存操作程序
+ /system：相当于电脑C盘，存放Android系统和系统应用
+ /recover: 回复分区。可以进入该分区进行系统回复
+ /data: 用户数据区，包含了用户的数据：联系人、短信、设置、用户安装的程序
+ /cache: 安卓系统缓存区，保存系统经常访问的数据和应用程序
+ /misc: 杂项内容
+ /sdcard: 用户自己的存储区域。存放照片视频等
Android系统启动跟PC相似。当开机时，首先加载BootLoader，BootLoader会读取ROM找到系统并将内核加载进RAM中。

当内核启动后会初始化各种软硬件环境，加载驱动程序，挂载跟文件系统。最后阶段会启动执行第一个用户空间进程init进程。
![](./imgs/startup.jpg)

+  参考资料[《详解Android系统启动过程》](http://www.ay1.cc/article/18368.html)

##### 3 你知道怎么添加一个系统服务吗？
###### 新增的服务
+ 服务AIDL文件，定义服务的接口：
    ```shell
    frameworks/base/core/java/android/app/IDemoManager.aidl
    ```
+ 服务管理类，提供给客户端调用，以访问服务端的接口 (即持有服务端的引用， Binder 引用)
    ```shell
    frameworks/base/core/java/android/app/DemoManager.java
    ```
+ 服务实现类， 实现AIDL文件
    ```shell
    frameworks/base/services/core/java/com/android/server/DemoManagerService.java
    ```

###### 创建及启动服务涉及的修改
+ 定义服务的标识
    ```shell
    frameworks/base/core/java/android/content/Context.java
    ```
+ 创建及启动服务
    ```shell
    frameworks/base/services/java/com/android/server/SystemServer.java
    ```
+ 创建服务管理类(可同时获取服务端的代理对象，由服务管理对象持有)
    ```shell
    frameworks/base/core/java/android/app/SystemServiceRegistry.java
    ```
+ 参考[《Framework添加新的系统服务》](https://www.jianshu.com/p/74971ee85a8b)

#### 4 系统服务和bind的应用服务有什么区别？
##### 启动方式
+ 系统服务
  在SystemServer里面进行分批、分阶段启动，大部分都跑在binder线程里面。
##### 注册方式
+ 系统服务
  有系统服务才能注册在ServiceManager
+ 应用服务
  ![](./imgs/ams.png)
    + 应用端会向AMS发起bindService。
    + AMS会先判断这个Service是否已经注册过了，注册过就直接把之前发布的binder返回给应用；如果没有，AMS会像Service请求binder对象。（AMS请求的，属于被动注册）
    + Service会相应AMS的请求，发布这个binder对象到AMS
    + AMS再把这个binder对象回调给应用

##### 使用方式
+ 系统服务
  通过服务名去找到对于的ServiceFetcher对象，然后先通过SM.getService拿到binder对象，然后封装了一层拿到服务的管理对象。
+ 应用服务
  通过bindService向AMS发送绑定服务端请求，AMS通过onServiceConnected()回调把服务的binder对象返回给业务端，然后把这个对象封装成业务接口对象给业务接口调用。

#### 5 ServiceManager启动和工作原理是怎样的？
ServiceManager 是为了完成 Binder Server 的 Name（域名）和 Handle（IP 地址）之间对应关系的查询而存在的，它主要包含的功能：

+ 注册：当一个 Binder Server 创建后，应该将这个 Server 的 Name 和 Handle 对应关系记录到 ServiceManager 中
+ 查询：其他应用可以根据 Server 的 Name 查询到对应的 Service Handle
```shell    
binder 驱动 -> 路由器
ServiceManager -> DNS
Binder Client -> 客户端
Binder Server -> 服务器
```

+ 参考资料[《ServiceManager 的工作原理》](https://zhuanlan.zhihu.com/p/158623349)

#### 6 谈谈对AMS的理解
AMS即ActivityManagerService，AMS是Android中最核心的服务，主要负责系统中四大组件的启动、切换、调度及应用进程的管理和调度等工作，其职责与操作系统中的进程管理和调度模块相类似
![](./imgs/ams_service.jpeg)

  + AMS的main函数：创建AMS实例，其中最重要的工作是创建Android运行环境，得到一个ActivityThread和一个Context对象。
  + AMS的setSystemProcess函数：该函数注册AMS和meminfo等服务到ServiceManager中。另外，它为SystemServer创建了一个ProcessRecord对象。由于AMS是Java世界的进程管理及调度中心，要做到对Java进程一视同仁，尽管SystemServer贵为系统进程，此时也不得不将其并入AMS的管理范围内。
  + AMS的installSystemProviders：为SystemServer加载SettingsProvider。
  + AMS的systemReady：做系统启动完毕前最后一些扫尾工作。该函数调用完毕后，HomeActivity将呈现在用户面前。 

+ 参考资料[《AMS在Android起到什么作用?》](https://zhuanlan.zhihu.com/p/86266649)
  
### 第2章 应用进程相关面试问题

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
