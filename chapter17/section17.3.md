# 5.3 NSThread


NSThread是基于线程使用，轻量级的多线程编程方法（相对GCD和NSOperation），一个NSThread对象代表一个线程，需要手动管理线程的生命周期，处理线程同步等问题。


### 创建线程

* 方法一

```
// 1. 创建线程
 NSThread *thread = [[NSThread alloc] initWithTarget:self selector:@selector(test:) object:nil];
    thread.name = @"thread1"; //设置线程名
    [thread start];   // 2. 启动线程，此方法需要我们手动开启线程
- (void)test:(NSString *)string {
  NSLog(@"test - %@ - %@", [NSThread currentThread], string);
}

2017-09-29 10:28:44.203914+0800 aegewgr[9577:3142906] test - <NSThread: 0x600000461a40>{number = 3, name = thread1} - (null)

```


* 方法二  自动创建一个子线程，并在子线程中执行

```
[NSThread detachNewThreadSelector:@selector(test:) toTarget:self withObject:@"分离子线程"];


2017-09-29 10:33:21.702512+0800 aegewgr[9617:3159015] test - <NSThread: 0x6000004621c0>{number = 4, name = (null)} - 分离子线程

```

* 方法三

该方法会开启一条后台线程，并在后台线程中执行。
上面所有的方法都还有与之对应的通过block创建的方法。

```
[self performSelectorInBackground:@selector(test:) withObject:@"后台线程"];

```


###  通过线程间通信的几个方法

```
[self performSelectorOnMainThread:<#(nonnull SEL)#> withObject:<#(nullable id)#> waitUntilDone:<#(BOOL)#>]
[self performSelectorOnMainThread:<#(nonnull SEL)#> withObject:<#(nullable id)#> waitUntilDone:<#(BOOL)#> modes:<#(nullable NSArray<NSString *> *)#>]

[self performSelector:<#(nonnull SEL)#> onThread:<#(nonnull NSThread *)#> withObject:<#(nullable id)#> waitUntilDone:<#(BOOL)#>]
[self performSelector:<#(nonnull SEL)#> onThread:<#(nonnull NSThread *)#> withObject:<#(nullable id)#> waitUntilDone:<#(BOOL)#> modes:<#(nullable NSArray<NSString *> *)#>]


```


### NSThread的类方法

* 返回当前线程

```
// 当前线程
[NSThread currentThread];
NSLog(@"%@",[NSThread currentThread]);
 
// 如果number=1，则表示在主线程，否则是子线程
打印结果：{number = 1, name = main}

```

* 阻塞休眠

```
//休眠多久
[NSThread sleepForTimeInterval:2];
//休眠到指定时间
[NSThread sleepUntilDate:[NSDate date]];

```

* 类方法补充

```
//退出线程
[NSThread exit];
//判断当前线程是否为主线程
[NSThread isMainThread];
//判断当前线程是否是多线程
[NSThread isMultiThreaded];
//主线程的对象
NSThread *mainThread = [NSThread mainThread];

```




* NSThread的一些属性

```
//线程是否在执行
thread.isExecuting;
//线程是否被取消
thread.isCancelled;
//线程是否完成
thread.isFinished;
//是否是主线程
thread.isMainThread;
//线程的优先级，取值范围0.0到1.0，默认优先级0.5，1.0表示最高优先级，优先级高，CPU调度的频率高
 thread.threadPriority;

```




















