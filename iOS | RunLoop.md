---
title: iOS | RunLoop
date: 2020-07-20 20:28:12
categories:
    - iOS
tags: 
    - iOS
mathjax: true
---

## RunLoop的作用
RunLoop 实际上就是一个对象，这个对象管理了其需要处理的事件和消息，并提供了一个入口函数来执行上面 Event Loop 的逻辑。线程执行了这个函数后，就会一直处于这个函数内部 “接受消息->等待->处理” 的循环中，直到这个循环结束（比如传入 quit 的消息），函数返回。

## RunLoop与线程之间的关系
* RunLoop和线程之间是一一对应的，它们之间的关系保存在一个全局字典以及线程私有数据中；
* 在线程创建的时候，是没有对应的RunLoop，它的创建是在第一次获取的时候，它的销毁则发生在线程销毁的时候；

## NSTimer在列表滑动时失效的原因和解决方法
NSTimer默认运行在RunLoop的kCFRunLoopDefaultMode下，在列表滑动的时候，RunLoop会切换UITrackingRunLoopMode，因为RunLoop只能运行在一种模式下，所以NSTimer不会执行回调。
使用[[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];将timer添加到CommonModes中。它可以让timer运行在所有被标记为Common属性的Mode中，kCFRunLoopDefaultMode和UITrackingRunLoopMode默认都已经被标为”Common”属性的。

或者是默认执行的是主线程，但是timer设置在其他线程，并没有被调用

## NSTimer不精准的理由
NSTimer定时器的触发是基于RunLoop运行的，所以使用NSTimer之前必须注册到RunLoop，但是RunLoop为了节省资源并不会在非常准确的时间点调用定时器。当RunLoop在处理比较复杂的任务时，可能会错过一个时间点，那么定时器只能等到下一个时间点执行，并不会延后执行。

**NSTimer不精准的解决方法**
* 使用GCDTimer作为定时器，它是基于硬件时间的，相对来说更精准一点；
* 开辟子线程中进行NSTimer的操作，再在主线程中修改UI界面显示操作结果，这么做相对于将NSTimer添加在主线程RunLoop来说是会提高精确度，但是如果说子线程的RunLoop也比较繁忙的话，那一样会带来误差；

## RunLoop的mode和Common mode
Mode是RunLoop的运行模式，每个RunLoop需要运行在一个特定的Mode中。如果需要切换Mode，只能退出Loop，再重新指定一个Mode进入。不同组的source0/source1/observer/timer可以相互隔离，互不影响，从而提高执行效率。
常用的RunLoop的Mode
* kCFRunLoopDefaultMode：App的默认Mode，通常主线程是在这个Mode下运行；
* UITrackingRunLoopMode：界面跟踪Mode，用于滚动视图追踪触摸滑动，保证界面滑动时不受其他 Mode影响；
* kCFRunLoopCommonModes：这是一个占位用的Mode，并不是一种真正的Mode；
Common mode并不是一个具体的Mode，它只是用来将Mode标记为Common属性。当source0/source1/observer/timer被添加到Common mode下的时候，意味着他们可以运行在所有被标记为Common属性的Mode中。
kCFRunLoopDefaultMode和UITrackingRunLoopMode默认就被标记了Common属性。

一个 RunLoop 包含若干个 Mode，每个 Mode 又包含若干个 Source/Timer/Observer。每次调用 RunLoop 的主函数时，只能指定其中一个 Mode，这个Mode被称作 CurrentMode。如果需要切换 Mode，只能退出 Loop，再重新指定一个 Mode 进入。这样做主要是为了分隔开不同组的 Source/Timer/Observer，让其互不影响。
**CFRunLoopSourceRef** 是事件产生的地方。Source有两个版本：Source0 和 Source1。
* Source0 只包含了一个回调（函数指针），它并不能主动触发事件。使用时，你需要先调用 CFRunLoopSourceSignal(source)，将这个 Source 标记为待处理，然后手动调用 CFRunLoopWakeUp(runloop) 来唤醒 RunLoop，让其处理这个事件。
* Source1 包含了一个 mach_port 和一个回调（函数指针），被用于通过内核和其他线程相互发送消息。这种 Source 能主动唤醒 RunLoop 的线程，其原理在下面会讲到。
**CFRunLoopTimerRef** 是基于时间的触发器，它和 NSTimer 是toll-free bridged 的，可以混用。其包含一个时间长度和一个回调（函数指针）。当其加入到 RunLoop 时，RunLoop会注册对应的时间点，当时间点到时，RunLoop会被唤醒以执行那个回调。
**CFRunLoopObserverRef** 是观察者，每个 Observer 都包含了一个回调（函数指针），当 RunLoop 的状态发生变化时，观察者就能通过回调接受到这个变化。可以观测的时间点有以下几个：
```
typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
    kCFRunLoopEntry         = (1UL << 0), // 即将进入Loop
    kCFRunLoopBeforeTimers  = (1UL << 1), // 即将处理 Timer
    kCFRunLoopBeforeSources = (1UL << 2), // 即将处理 Source
    kCFRunLoopBeforeWaiting = (1UL << 5), // 即将进入休眠
    kCFRunLoopAfterWaiting  = (1UL << 6), // 刚从休眠中唤醒
    kCFRunLoopExit          = (1UL << 7), // 即将退出Loop
};
```
上面的 Source/Timer/Observer 被统称为 **mode item**，一个 item 可以被同时加入多个 mode。但一个 item 被重复加入同一个 mode 时是不会有效果的。如果一个 mode 中一个 item 都没有，则 RunLoop 会直接退出，不进入循环。

## RunLoop执行周期
[image:D51A08C5-2E5D-478D-80C5-F832A2436004-89533-00006FCAA6D3C017/截屏2020-07-20 上午11.08.15.png]

## RunLoop底层实现
为了实现消息的发送和接收，mach_msg() 函数实际上是调用了一个 Mach 陷阱 (trap)，即函数mach_msg_trap()，陷阱这个概念在 Mach 中等同于系统调用。当你在用户态调用 mach_msg_trap() 时会触发陷阱机制，切换到内核态；内核态中内核实现的 mach_msg() 函数会完成实际的工作，如下图：
[image:E153048B-62CF-4244-9231-9A6668CEC4FD-89533-0000705C7F4B3AC0/RunLoop_5.png]
RunLoop 的核心就是一个 mach_msg() ，RunLoop 调用这个函数去接收消息，如果没有别人发送 port 消息过来，内核会将线程置于等待状态。例如你在模拟器里跑起一个 iOS 的 App，然后在 App 静止时点击暂停，你会看到主线程调用栈是停留在 mach_msg_trap() 这个地方。


## RunLoop中Source0和Source1的区别**
Source0并不能主动触发事件。使用时，你需要先调用CFRunLoopSourceSignal，将这个Source标记为待处理，然后手动调用CFRunLoopWakeUp来唤醒RunLoop，让其处理这个事件。
Source1能主动触发事件。其中它有一个mach_port_t，mach_port是用于内核向线程发送消息的。
使用Source0的情况：
* 触摸事件处理；
* 调用performSelector:onThread:withObject:waitUntilDone:方法；
使用Source1的情况：
* 基于端口的线程间通信（A线程通过端口发送消息到B线程，这个消息是Source1的；
* 系统事件的捕捉，先触发是Source1，接着分发到Source0去处理。

## RunLoop响应用户操作
以按钮点击触发事件为例，点击屏幕的时候，首先系统内部捕获到这个点击事件，这是在Source1中处理的，Source1会包装成事件丢到事件队列中，交给Source0处理。

## RunLoop与UI刷新
当UI需要更新的时候，比如改变了frame、更新了UIView/CALayer的层次时，或者手动调用了setNeedsLayout/setNeedsDisplay方法后，这个UIView/CALayer就被标记为待处理，并被提交到一个全局的容器去。
苹果注册了一个Observer监听BeforeWaiting(即将进入休眠) 和 Exit(即将退出Loop) 事件，回调去执行一个很长的函数：CA::Transaction::observer_callback(__CFRunLoopObserver*, unsigned long, void*)() 这个函数里会遍历所有待处理的UIView/CAlayer以执行实际的绘制和调整，并更新界面。
[image:B7E29B3F-C4BA-41D9-808A-01428EF72063-89533-00006F8AFBC1A740/RunLoopUI刷新.jpg]

## RunLoop与AutoreleasePool
在程序启动之后，主线程会创建一个Runloop，也会创建两个Observer，回调工作都是在_wrapRunLoopWithAutoreleasePoolHandler函数中。
第一个Observer监听的是Entry（即将进入Loop），回调是在_objc_autoreleasePoolPush()中创建自动释放池的，优先级是最高的，保证创建释放池是在所有回调之前。
第二个Observer监听有两个事件：BeforeWaiting（进入休眠）时调用_objc_autoreleasePoolPop和_objc_autoreleasePoolPush释放旧的释放池以及创建新的释放池；Exit（退出Loop）调用_objc_autoreleasePoolPop来释放自动释放池。这个优先级是最低的，保证释放池发生在所有回调之后调用。
