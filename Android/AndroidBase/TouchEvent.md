### Touch事件机制
**touch事件的分发机制重点在于dispatchTouchEvent的返回值，true表示这个已经被消费了，false表示这个事件还没有被消费（不需要像许多网上的解释额外增加一个super调用的解释，增加了理解的困难，super就是默认的判断方式，具体如何判断具体分析）而重写的onTouchEvent的返回值是对应的，表示的就是这个事件是否被消费了**

##### Activity中的dispatchTouchEvent
首先会通过Window的superDispatchTouchEvent的方法调用，最终调用到DecorView的dispatchTouchEvent，这里就分发给了当前窗口顶端的ViewGroup，然后通过ViewGroup的实现一步一步分发给子View，这个事件被消费了，则返回true，当这些方法都没有消费这个事件，则最终分发到activity的onTouchEvent方法最终尝试消费这个事件，此时onTouchEvent返回的true/false表示这个是否被消费

##### ViewGroup的dispatchTouchEvent
首先调用onInterceptTouchEvent方法，确定是否需要中断分发，true表示需要中断，则调用当前的onTouchEvent方法（实际是调用父类View的super.dispatchTouchEvent()，然后才调用到onTouchEvent()方法）消费这个事件，并且将调用的结果返回；false表示不需要中断，则继续在包含的View中找到合适的对象调用dispatchTouchEvent()继续分发事件

##### View的dispatchTouchEvent
因为不会包含view，因此没有必要分发事件，直接调用View的onTouchEvent方法并返回相应结果

##### 核心流程伪代码：
```
Activity.dispatchTouchEvent() {
    result = ViewGroup.dispatchTouchEvent() {
        if (ViewGroup.onInterceptTouchEvent()) {
            return ViewGroup.onTouchEvent();
        } else {
            // 重复一遍ViewGroup调用
            return ViewGroupSub.dispatchTouchEvent() {
                if (ViewGroupSub.onInterceptTouchEvent()) {
                    return ViewGroupSub.onTouchEvent();
                } else {
                    if (this instanceof View) {
                        return this.onTouchEvent();
                    } else {
                        return SubSubViews.dispatchTouchEvent();
                    }
                }
            }
        }
    }
    if (!result) {
        result = Activity.onTouchEvent();
    }
    return result;
}
```
