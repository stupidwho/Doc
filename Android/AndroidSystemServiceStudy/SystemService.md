# Android 系统服务入门

### 简述
Android通过Service的方式提供各种各样的服务供应用使用，从而为应用开发提供多种多样的功能。通过service的方式进行服务的调用，包括最重要的ActivityManagerService、WindowManagerService等等内容，一般都是通过context.getServiceManager()获取服务的代理类，其真正的实现类就是XXXService。

### 服务基础Binder机制

Android系统是基于linux的系统，因此进程是Android系统中的一个总要内容。而各种不同的Service处于不同的进程当中，如果需要调用不同的服务，则需要跨进程去调用以及传递数据，而如何跨进程的去调用这些服务，并且高效地传递数据、通讯，则需要更深入地去分析。Android为了使得这个过程更为适合移动端，自己建立了一套基于Binder的IPC机制，因此我们要了解各种服务的调用，首先要从Android的跨进程通信的机制研究开始。

* Linux中跨进程的通信机制主要有管道方式、消息队列、共享内存、信号量、Socket等通信方式，从性能、安全性方面对比起来都没法满足移动端的需求

* Binder基于Client-Server通信模式，传输过程只需一次拷贝，为发送方添加UID/PID身份，既支持实名Binder也支持匿名Binder，安全性高。

### Binder机制简述

Binder作为系统服务中联系C层ManagerService和java层Manager的重要组件，有着举足轻重的作用。Binder引入了面向对象的思想，将进程间通信转化为通过对某个Binder对象的引用调用该对象的方法，而其独特之处在于Binder对象是一个可以跨进程引用的对象，从而使得系统服务在使用上十分的清晰、便利，同时性能方面也能够兼顾。

### Binder流程

整个流程包含4个内容，Client、Server、ServiceManager和Binder驱动，用户负责上层的Client、Server编程，通过ServiceManager进行通信与调用，而Binder驱动则是通信过程中的底层实现。

![Binder](./binder_img.gif)


##### 


##### 性能分析

数据拷贝次数：

共享内存：0次

Binder：1次

Socket、管道、消息队列：2次

##### 安全性

Binder会发送UID/PID等身份相关内容，支持实名或匿名Binder，而其它的通信机制都不包含这样的内容

##### 复杂性：

共享内存过于复杂，需要维护各种变量，使得使用十分困难，尽管性能最好

管道、消息队列等虽然比较简单，也是操作系统中经常用到的进程间通信方式，然而由于需要两次数据拷贝，因此仍旧不能满足

Binder机制是综合下来最适用的机制，只进行1次数据拷贝，而且引入面向对象的方法，使用更为简单



参考文档：http://blog.csdn.net/universus/article/details/6211589
