# Handler机制

## 逻辑概述

### MessageQueue

* MessageQueue链表结构保存消息队列（按时间排）
* enqueueMessage()：插入队列头需要唤醒，nativeWake()
* next()：for循环，从队列取一个Message，当前消息时间未到，则需要调用nativePollOnce(intervalTime)方法控制阻塞，时间已到则获取队头消息
* IdleHandler：当next()方法中for循环第一次循环中取消息没有取到，需要阻塞时，先调用IdleHandler.queueIdle()方法，即每次loop获取消息都有一次处理取不到消息阻塞的回调
* **nativePollOnce和nativeWake调用非常频繁，因此需要高效的方法睡眠唤醒，因此native层采用的是epoll机制（通过文件描述符的管道IO来控制阻塞）**

### Message

* 链表结构保存回收的Message，obtain方法从链表中获取消息
* target属性保存关联的Handler，callback属性保存关联的Runnable

### Looper

* loop：for循环调用MessageQueue.next()获取下一个消息，执行Message.target.handlerMessage方法，如果取到null，结束循环

## 同步屏障

* postSyncBarrier调用后，插入一个没有target的Message到消息队列中
* 设置屏障后，优先执行异步消息，同步消息暂不执行
* 同步屏障导致MessageQueue的管理变复杂，插入、获取都需要考虑屏障相关的兼容
* 目的：确保某些UI任务可以尽快执行，而不是执行完队列中的其他Message再执行
* 同步异步怎么理解：Handler原来是按照队列一个一个执行，是同步的，加入屏障后，普通的同步消息被绕过，优先执行了后面的消息，即是异步的
