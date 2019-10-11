# 5.6 NSOperation与 NSOperationQueue


NSOperation、NSOperationQueue 是苹果提供给我们的一套多线程解决方案。实际上 NSOperation、NSOperationQueue 是基于 GCD 更高一层的封装，完全面向对象。但是比 GCD 更简单易用、代码可读性也更高。

为什么要使用 NSOperation、NSOperationQueue？

* 可添加完成的代码块，在操作完成后执行。
* 添加操作之间的依赖关系，方便的控制执行顺序。
* 设定操作执行的优先级。
* 可以很方便的取消一个操作的执行。
* 使用 KVO 观察对操作执行状态的更改：isExecuteing、isFinished、isCancelled。

既然是基于 GCD 的更高一层的封装。那么，GCD 中的一些概念同样适用于 NSOperation、NSOperationQueue。在 NSOperation、NSOperationQueue 中也有类似的任务（操作）和队列（操作队列）的概念。

* 操作（Operation）：
	* 执行操作的意思，换句话说就是你在线程中执行的那段代码。
	* 在 GCD 中是放在 block 中的。在 NSOperation 中，我们使用 NSOperation 子类 NSInvocationOperation、NSBlockOperation，或者自定义子类来封装操作。

* 操作队列（Operation Queues）：
	* 这里的队列指操作队列，即用来存放操作的队列。不同于 GCD 中的调度队列 FIFO（先进先出）的原则。NSOperationQueue 对于添加到队列中的操作，首先进入准备就绪的状态（就绪状态取决于操作之间的依赖关系），然后进入就绪状态的操作的开始执行顺序（非结束执行顺序）由操作之间相对的优先级决定（优先级是操作对象自身的属性）。
	* 操作队列通过设置最大并发操作数（maxConcurrentOperationCount）来控制并发、串行。
	* NSOperationQueue 为我们提供了两种不同类型的队列：主队列和自定义队列。主队列运行在主线程之上，而自定义队列在后台执行。



##  NSOperation的理解与使用

### NSOperation简介

NSOperation需要配合NSOperationQueue来实现多线程。因为默认情况下，NSOperation单独使用时系统同步执行操作，并没有开辟新线程的能力，只有配合NSOperationQueue才能实现异步执行。

因为NSOperation是基于GCD的，那么使用起来也和GCD差不多，其中，NSOperation相当于GCD中的任务，而NSOperationQueue则相当于GCD中的队列。NSOperation实现多线程的使用步骤分为三步：

1. 创建任务：先将需要执行的操作封装到一个NSOperation对象中。
2. 创建队列：创建NSOperationQueue对象。
3. 将任务加入到队列中：然后将NSOperation对象添加到NSOperationQueue中。
之后呢，系统就会自动将NSOperationQueue中的NSOperation取出来，在新线程中执行操作。


NSOperation 是个抽象类，不能用来封装操作。我们只有使用它的子类来封装操作。我们有三种方式来封装操作。

* 使用子类 NSInvocationOperation
* 使用子类 NSBlockOperation
* 自定义继承自 NSOperation 的子类，通过实现内部相应的方法来封装操作。


#### 使用子类 NSInvocationOperation

```
/**
 * 使用子类 NSInvocationOperation
 */
- (void)useInvocationOperation {

    // 1.创建 NSInvocationOperation 对象
    NSInvocationOperation *op = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(task1) object:nil];

    // 2.调用 start 方法开始执行操作
    [op start];
}

/**
 * 任务1
 */
- (void)task1 {
    for (int i = 0; i < 2; i++) {
        [NSThread sleepForTimeInterval:2]; // 模拟耗时操作
        NSLog(@"1---%@", [NSThread currentThread]); // 打印当前线程
    }
}


```

* 可以看到：在没有使用 NSOperationQueue、在主线程中单独使用使用子类 NSInvocationOperation 执行一个操作的情况下，操作是在当前线程执行的，并没有开启新线程。


如果在其他线程中执行操作，则打印结果为其他线程。

```
// 在其他线程使用子类 NSInvocationOperation
[NSThread detachNewThreadSelector:@selector(useInvocationOperation) toTarget:self withObject:nil];

```

* 可以看到：在其他线程中单独使用子类 NSInvocationOperation，操作是在当前调用的其他线程执行的，并没有开启新线程。


### 使用子类 NSBlockOperation

```
/**
 * 使用子类 NSBlockOperation
 */
- (void)useBlockOperation {

    // 1.创建 NSBlockOperation 对象
    NSBlockOperation *op = [NSBlockOperation blockOperationWithBlock:^{
        for (int i = 0; i < 2; i++) {
            [NSThread sleepForTimeInterval:2]; // 模拟耗时操作
            NSLog(@"1---%@", [NSThread currentThread]); // 打印当前线程
        }
    }];

    // 2.调用 start 方法开始执行操作
    [op start];
}


```

* 可以看到：在没有使用 NSOperationQueue、在主线程中单独使用 NSBlockOperation 执行一个操作的情况下，操作是在当前线程执行的，并没有开启新线程。
* 注意：和上边 NSInvocationOperation 使用一样。因为代码是在主线程中调用的，所以打印结果为主线程。如果在其他线程中执行操作，则打印结果为其他线程。


但是，NSBlockOperation 还提供了一个方法 ``addExecutionBlock:``，通过 ``addExecutionBlock:`` 就可以为 NSBlockOperation 添加额外的操作。这些操作（包括 ``blockOperationWithBlock`` 中的操作）可以在不同的线程中同时（并发）执行。只有当所有相关的操作已经完成执行时，才视为完成。

如果添加的操作多的话，``blockOperationWithBlock:`` 中的操作也可能会在其他线程（非当前线程）中执行，这是由系统决定的，并不是说添加到 ``blockOperationWithBlock:`` 中的操作一定会在当前线程中执行。（可以使用 ``addExecutionBlock: ``多添加几个操作试试）。


```
/**
 * 使用子类 NSBlockOperation
 * 调用方法 AddExecutionBlock:
 */
- (void)useBlockOperationAddExecutionBlock {

    // 1.创建 NSBlockOperation 对象
    NSBlockOperation *op = [NSBlockOperation blockOperationWithBlock:^{
        for (int i = 0; i < 2; i++) {
            [NSThread sleepForTimeInterval:2]; // 模拟耗时操作
            NSLog(@"1---%@", [NSThread currentThread]); // 打印当前线程
        }
    }];

    // 2.添加额外的操作
    [op addExecutionBlock:^{
        for (int i = 0; i < 2; i++) {
            [NSThread sleepForTimeInterval:2]; // 模拟耗时操作
            NSLog(@"2---%@", [NSThread currentThread]); // 打印当前线程
        }
    }];
    [op addExecutionBlock:^{
        for (int i = 0; i < 2; i++) {
            [NSThread sleepForTimeInterval:2]; // 模拟耗时操作
            NSLog(@"3---%@", [NSThread currentThread]); // 打印当前线程
        }
    }];
    [op addExecutionBlock:^{
        for (int i = 0; i < 2; i++) {
            [NSThread sleepForTimeInterval:2]; // 模拟耗时操作
            NSLog(@"4---%@", [NSThread currentThread]); // 打印当前线程
        }
    }];
    [op addExecutionBlock:^{
        for (int i = 0; i < 2; i++) {
            [NSThread sleepForTimeInterval:2]; // 模拟耗时操作
            NSLog(@"5---%@", [NSThread currentThread]); // 打印当前线程
        }
    }];
    [op addExecutionBlock:^{
        for (int i = 0; i < 2; i++) {
            [NSThread sleepForTimeInterval:2]; // 模拟耗时操作
            NSLog(@"6---%@", [NSThread currentThread]); // 打印当前线程
        }
    }];
    [op addExecutionBlock:^{
        for (int i = 0; i < 2; i++) {
            [NSThread sleepForTimeInterval:2]; // 模拟耗时操作
            NSLog(@"7---%@", [NSThread currentThread]); // 打印当前线程
        }
    }];
    [op addExecutionBlock:^{
        for (int i = 0; i < 2; i++) {
            [NSThread sleepForTimeInterval:2]; // 模拟耗时操作
            NSLog(@"8---%@", [NSThread currentThread]); // 打印当前线程
        }
    }];

    // 3.调用 start 方法开始执行操作
    [op start];
}


```

输出结果：

```
2018-11-11 16:33:21.693479+0800 AppLe[93490:4268504] 6---<NSThread: 0x60c0002708c0>{number = 7, name = (null)}
2018-11-11 16:33:21.693482+0800 AppLe[93490:4268366] 3---<NSThread: 0x60c000261080>{number = 6, name = (null)}
2018-11-11 16:33:21.693481+0800 AppLe[93490:4268369] 4---<NSThread: 0x600000265100>{number = 5, name = (null)}
2018-11-11 16:33:21.693481+0800 AppLe[93490:4268503] 7---<NSThread: 0x608000467240>{number = 8, name = (null)}
2018-11-11 16:33:21.693509+0800 AppLe[93490:4268505] 8---<NSThread: 0x60c000270700>{number = 9, name = (null)}
2018-11-11 16:33:21.693511+0800 AppLe[93490:4268310] 5---<NSThread: 0x60000007cbc0>{number = 1, name = main}
2018-11-11 16:33:21.693512+0800 AppLe[93490:4268367] 1---<NSThread: 0x608000274380>{number = 3, name = (null)}
2018-11-11 16:33:21.693518+0800 AppLe[93490:4268370] 2---<NSThread: 0x60400007cdc0>{number = 4, name = (null)}
2018-11-11 16:33:23.695127+0800 AppLe[93490:4268366] 3---<NSThread: 0x60c000261080>{number = 6, name = (null)}
2018-11-11 16:33:23.695123+0800 AppLe[93490:4268369] 4---<NSThread: 0x600000265100>{number = 5, name = (null)}
2018-11-11 16:33:23.695123+0800 AppLe[93490:4268504] 6---<NSThread: 0x60c0002708c0>{number = 7, name = (null)}
2018-11-11 16:33:23.695134+0800 AppLe[93490:4268370] 2---<NSThread: 0x60400007cdc0>{number = 4, name = (null)}
2018-11-11 16:33:23.695144+0800 AppLe[93490:4268310] 5---<NSThread: 0x60000007cbc0>{number = 1, name = main}
2018-11-11 16:33:23.695134+0800 AppLe[93490:4268503] 7---<NSThread: 0x608000467240>{number = 8, name = (null)}
2018-11-11 16:33:23.695123+0800 AppLe[93490:4268505] 8---<NSThread: 0x60c000270700>{number = 9, name = (null)}
2018-11-11 16:33:23.695153+0800 AppLe[93490:4268367] 1---<NSThread: 0x608000274380>{number = 3, name = (null)}

```



* 可以看出：使用子类 NSBlockOperation，并调用方法 AddExecutionBlock: 的情况下，blockOperationWithBlock:方法中的操作 和 addExecutionBlock: 中的操作是在不同的线程中异步执行的。而且，这次执行结果中 blockOperationWithBlock:方法中的操作也不是在当前线程（主线程）中执行的。从而印证了blockOperationWithBlock: 中的操作也可能会在其他线程（非当前线程）中执行。

一般情况下，如果一个 NSBlockOperation 对象封装了多个操作。NSBlockOperation 是否开启新线程，取决于操作的个数。如果添加的操作的个数多，就会自动开启新线程。当然开启的线程数是由系统来决定的。


###  使用自定义继承自 NSOperation 的子类

如果使用子类 NSInvocationOperation、NSBlockOperation 不能满足日常需求，我们可以使用自定义继承自 NSOperation 的子类。可以通过重写 main 或者 start 方法 来定义自己的 NSOperation 对象。重写main方法比较简单，我们不需要管理操作的状态属性 isExecuting 和 isFinished。当 main 执行完返回的时候，这个操作就结束了。

先定义一个继承自 NSOperation 的子类，重写main方法。

```
// YSCOperation.h 文件
#import <Foundation/Foundation.h>

@interface YSCOperation : NSOperation

@end

// YSCOperation.m 文件
#import "YSCOperation.h"

@implementation YSCOperation

- (void)main {
    if (!self.isCancelled) {
        for (int i = 0; i < 2; i++) {
            [NSThread sleepForTimeInterval:2];
            NSLog(@"1---%@", [NSThread currentThread]);
        }
    }
}

@end


```

然后使用的时候导入头文件YSCOperation.h。

```
/**
 * 使用自定义继承自 NSOperation 的子类
 */
- (void)useCustomOperation {
    // 1.创建 YSCOperation 对象
    YSCOperation *op = [[YSCOperation alloc] init];
    // 2.调用 start 方法开始执行操作
    [op start];
}

--0---<NSThread: 0x6080000779c0>{number = 1, name = main}
--1---<NSThread: 0x6080000779c0>{number = 1, name = main}

```

* 可以看出：在没有使用 NSOperationQueue、在主线程单独使用自定义继承自 NSOperation 的子类的情况下，是在主线程执行操作，并没有开启新线程。



## 队列NSOperationQueue


NSOperationQueue只有两种队列：主队列、其他队列。其他队列包含了串行和并发。

主队列的创建如下，主队列上的任务是在主线程执行的。

```
NSOperationQueue *mainQueue = [NSOperationQueue mainQueue];

```


其他队列（非主队列）的创建如下，加入到‘非队列’中的任务默认就是并发，开启多线程。

```
NSOperationQueue *queue = [[NSOperationQueue alloc] init];

```


注意：

1. 非主队列（其他队列）可以实现串行或并行。
2. 队列NSOperationQueue有一个参数叫做最大并发数：maxConcurrentOperationCount。
3. maxConcurrentOperationCount默认为-1，直接并发执行，所以加入到‘非队列’中的任务默认就是并发，开启多线程。
4. 当maxConcurrentOperationCount为1时，则表示不开线程，也就是串行。
5. 当maxConcurrentOperationCount大于1时，进行并发执行。
6. 系统对最大并发数有一个限制，所以即使程序员把maxConcurrentOperationCount设置的很大，系统也会自动调整。所以把最大并发数设置的很大是没有意义的。


#### NSOperation + NSOperationQueue


* 把任务加入队列，这才是NSOperation的常规使用方式。

	先创建好任务，然后运用- (void)addOperation:(NSOperation *)op 方法来吧任务添加到队列中，示例代码如下：

```
- (void)testOperationQueue {
    // 创建队列，默认并发
    NSOperationQueue *queue = [[NSOperationQueue alloc] init];
 
    // 创建操作，NSInvocationOperation
    NSInvocationOperation *invocationOperation = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(invocationOperationAddOperation) object:nil];
    // 创建操作，NSBlockOperation
    NSBlockOperation *blockOperation = [NSBlockOperation blockOperationWithBlock:^{
        for (int i = 0; i < 3; i++) {
            NSLog(@"addOperation把任务添加到队列======%@", [NSThread currentThread]);
        }
    }];
 
    [queue addOperation:invocationOperation];
    [queue addOperation:blockOperation];
}
 
 
- (void)invocationOperationAddOperation {
    NSLog(@"invocationOperation===aaddOperation把任务添加到队列====%@", [NSThread currentThread]);
}

```

运行结果如下，可以看出，任务都是在子线程执行的，开启了新线程！

```
invocationOperation===addOperation把任务添加到队列===={number = 4, name = (null)}
addOperation把任务添加到队列======{number = 3, name = (null)}
addOperation把任务添加到队列======{number = 3, name = (null)}
addOperation把任务添加到队列======{number = 3, name = (null)}


```


* addOperationWithBlock添加任务到队列

  这是一个更方便的把任务添加到队列的方法，直接把任务写在block中，添加到任务中。


```
- (void)testAddOperationWithBlock {
    // 创建队列，默认并发
    NSOperationQueue *queue = [[NSOperationQueue alloc] init];
 
    // 添加操作到队列
    [queue addOperationWithBlock:^{
        for (int i = 0; i < 3; i++) {
            NSLog(@"addOperationWithBlock把任务添加到队列======%@", [NSThread currentThread]);
        }
    }];
}


addOperationWithBlock把任务添加到队列======{number = 3, name = (null)}
addOperationWithBlock把任务添加到队列======{number = 3, name = (null)}
addOperationWithBlock把任务添加到队列======{number = 3, name = (null)}


```



* 运用最大并发数实现串行

	可以运用队列的属性maxConcurrentOperationCount（最大并发数）来实现串行，值需要把它设置为1就可以了，下面我们通过代码验证一下。
	
	NSOperation、NSOperationQueue 最吸引人的地方是它能添加操作之间的依赖关系。通过操作依赖，我们可以很方便的控制操作之间的执行先后顺序。NSOperation 提供了3个接口供我们管理和查看依赖。

	* ``- (void)addDependency:(NSOperation *)op``; 添加依赖，使当前操作依赖于操作 op 的完成。
	* ``- (void)removeDependency:(NSOperation *)op``; 移除依赖，取消当前操作对操作 op 的依赖。
	* ``@property (readonly, copy) NSArray<NSOperation *> *dependencies``; 在当前操作开始执行之前完成执行的所有操作对象数组。



```
- (void)testMaxConcurrentOperationCount {
    // 创建队列，默认并发
    NSOperationQueue *queue = [[NSOperationQueue alloc] init];
 
    // 最大并发数为1，串行
    queue.maxConcurrentOperationCount = 1;
 
    // 最大并发数为2，并发
//    queue.maxConcurrentOperationCount = 2;
 
 
    // 添加操作到队列
    [queue addOperationWithBlock:^{
        for (int i = 0; i < 3; i++) {
            NSLog(@"addOperationWithBlock把任务添加到队列1======%@", [NSThread currentThread]);
        }
    }];
    // 添加操作到队列
    [queue addOperationWithBlock:^{
        for (int i = 0; i < 3; i++) {
            NSLog(@"addOperationWithBlock把任务添加到队列2======%@", [NSThread currentThread]);
        }
    }];
 
    // 添加操作到队列
    [queue addOperationWithBlock:^{
        for (int i = 0; i < 3; i++) {
            NSLog(@"addOperationWithBlock把任务添加到队列3======%@", [NSThread currentThread]);
        }
    }];
}


addOperationWithBlock把任务添加到队列1======{number = 3, name = (null)}
addOperationWithBlock把任务添加到队列1======{number = 3, name = (null)}
addOperationWithBlock把任务添加到队列1======{number = 3, name = (null)}
addOperationWithBlock把任务添加到队列2======{number = 3, name = (null)}
addOperationWithBlock把任务添加到队列2======{number = 3, name = (null)}
addOperationWithBlock把任务添加到队列2======{number = 3, name = (null)}
addOperationWithBlock把任务添加到队列3======{number = 3, name = (null)}
addOperationWithBlock把任务添加到队列3======{number = 3, name = (null)}
addOperationWithBlock把任务添加到队列3======{number = 3, name = (null)}



```

当最大并发数为1的时候，虽然开启了线程，但是任务是顺序执行的，所以实现了串行。


### NSOperationQueue的其他操作

* 取消队列NSOperationQueue的所有操作，NSOperationQueue对象方法

```
- (void)cancelAllOperations

```

* 取消NSOperation的某个操作，NSOperation对象方法

```
- (void)cancel

```

* 使队列暂停或继续

```
// 暂停队列
[queue setSuspended:YES];

```

* 判断队列是否暂停

```
- (BOOL)isSuspended

```


###  NSOperation 优先级



NSOperation 提供了``queuePriority``（优先级）属性，queuePriority属性适用于同一操作队列中的操作，不适用于不同操作队列中的操作。默认情况下，所有新创建的操作对象优先级都是``NSOperationQueuePriorityNormal``。但是我们可以通过setQueuePriority:方法来改变当前操作在同一队列中的执行优先级。

```
// 优先级的取值
typedef NS_ENUM(NSInteger, NSOperationQueuePriority) {
    NSOperationQueuePriorityVeryLow = -8L,
    NSOperationQueuePriorityLow = -4L,
    NSOperationQueuePriorityNormal = 0,
    NSOperationQueuePriorityHigh = 4,
    NSOperationQueuePriorityVeryHigh = 8
};

public enum NSOperationQueuePriority : Int {  
    case VeryLow
    case Low
    case Normal
    case High
    case VeryHigh
}

```

需要注意的是，NSOperationQueue 也不能完全保证优先级高的任务一定先执行。
queuePriority默认值是NSOperationQueuePriorityNormal。根据实际需要我们可以通过调用queuePriority的setter方法修改某个操作的优先级。

```
NSBlockOperation *blkop1 = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"执行blkop1");
    }];

    NSBlockOperation *blkop2 = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"执行blkop2");
    }];

    // 设置操作优先级
    blkop1.queuePriority = NSOperationQueuePriorityLow;
    blkop2.queuePriority = NSOperationQueuePriorityVeryHigh;

    NSLog(@"blkop1 == %@",blkop1);
    NSLog(@"blkop2 == %@",blkop2);

    NSOperationQueue *queue = [[NSOperationQueue alloc] init];
    // 操作添加到队列
    [queue addOperation:blkop1];
    [queue addOperation:blkop2];

    NSLog(@"%@",[queue operations]);
    for (NSOperation *op in [queue operations]) {
        NSLog(@"op == %@",op);
    }

// 输出结果：
2017-02-12 19:36:01.149 NSOperation[1712:177976] blkop1 == <NSBlockOperation: 0x608000044440>
2017-02-12 19:36:01.150 NSOperation[1712:177976] blkop2 == <NSBlockOperation: 0x6080000444d0>
2017-02-12 19:36:01.150 NSOperation[1712:177976] (
    "<NSBlockOperation: 0x608000044440>",
    "<NSBlockOperation: 0x6080000444d0>"
)
2017-02-12 19:36:01.150 NSOperation[1712:177976] op == <NSBlockOperation: 0x608000044440>
2017-02-12 19:36:01.150 NSOperation[1712:177976] op == <NSBlockOperation: 0x6080000444d0>
2017-02-12 19:36:01.150 NSOperation[1712:178020] 执行blkop1
2017-02-12 19:36:01.151 NSOperation[1712:178021] 执行blkop2


```


解析：

1. 上面创建了两个blockOperation并且分别设置了优先级。显然blkop1的优先级低于blkop2的优先级。然后调用了队列的addOperation:方法使操作入队。最后输出结果证明，操作在对列中的顺去取决于addOperation:方法而不是优先级。
2. 虽然blkop2优先级高于blkop1，但是bloop1却先于blkop2执行完成。所以，优先级高的操作不一定先执行完成。




## NSOperation、NSOperationQueue 非线程安全

线程安全解决方案：可以给线程加锁，在一个线程执行该操作的时候，不允许其他线程进行操作。iOS 实现线程加锁有很多种方式。@synchronized、 NSLock、NSRecursiveLock、NSCondition、NSConditionLock、pthread_mutex、dispatch_semaphore、OSSpinLock、atomic(property) set/ge等等各种方式。这里我们使用 NSLock 对象来解决线程同步问题。NSLock 对象可以通过进入锁时调用 lock 方法，解锁时调用 unlock 方法来保证线程安全。

考虑线程安全的代码：

```
/**
 * 线程安全：使用 NSLock 加锁
 * 初始化火车票数量、卖票窗口(线程安全)、并开始卖票
 */

- (void)initTicketStatusSave {
    NSLog(@"currentThread---%@",[NSThread currentThread]); // 打印当前线程

    self.ticketSurplusCount = 50;

    self.lock = [[NSLock alloc] init];  // 初始化 NSLock 对象

    // 1.创建 queue1,queue1 代表北京火车票售卖窗口
    NSOperationQueue *queue1 = [[NSOperationQueue alloc] init];
    queue1.maxConcurrentOperationCount = 1;

    // 2.创建 queue2,queue2 代表上海火车票售卖窗口
    NSOperationQueue *queue2 = [[NSOperationQueue alloc] init];
    queue2.maxConcurrentOperationCount = 1;

    // 3.创建卖票操作 op1
    NSBlockOperation *op1 = [NSBlockOperation blockOperationWithBlock:^{
        [self saleTicketSafe];
    }];

    // 4.创建卖票操作 op2
    NSBlockOperation *op2 = [NSBlockOperation blockOperationWithBlock:^{
        [self saleTicketSafe];
    }];

    // 5.添加操作，开始卖票
    [queue1 addOperation:op1];
    [queue2 addOperation:op2];
}

/**
 * 售卖火车票(线程安全)
 */
- (void)saleTicketSafe {
    while (1) {

        // 加锁
        [self.lock lock];

        if (self.ticketSurplusCount > 0) {
            //如果还有票，继续售卖
            self.ticketSurplusCount--;
            NSLog(@"%@", [NSString stringWithFormat:@"剩余票数:%d 窗口:%@", self.ticketSurplusCount, [NSThread currentThread]]);
            [NSThread sleepForTimeInterval:0.2];
        }

        // 解锁
        [self.lock unlock];

        if (self.ticketSurplusCount <= 0) {
            NSLog(@"所有火车票均已售完");
            break;
        }
    }
}


```



在考虑了线程安全，使用 NSLock 加锁、解锁机制的情况下，得到的票数是正确的，没有出现混乱的情况。我们也就解决了多个线程同步的问题。


## NSOperation、NSOperationQueue 常用属性和方法归纳

### NSOperation 常用属性和方法


1. 取消操作方法
	* ``- (void)cancel; ``可取消操作，实质是标记 isCancelled 状态。

2. 判断操作状态方法
	* ``- (BOOL)isFinished; ``判断操作是否已经结束。
	* ``- (BOOL)isCancelled;`` 判断操作是否已经标记为取消。
	* ``- (BOOL)isExecuting;`` 判断操作是否正在在运行。
	* ``- (BOOL)isReady;`` 判断操作是否处于准备就绪状态，这个值和操作的依赖关系相关。
3. 操作同步
	* ``- (void)waitUntilFinished;`` 阻塞当前线程，直到该操作结束。可用于线程执行顺序的同步。
	* ``- (void)setCompletionBlock:(void (^)(void))block; completionBlock`` 会在当前操作执行完毕时执行 completionBlock。
	* ``- (void)addDependency:(NSOperation *)op;`` 添加依赖，使当前操作依赖于操作 op 的完成。
	* ``- (void)removeDependency:(NSOperation *)op;`` 移除依赖，取消当前操作对操作 op 的依赖。
	* ``@property (readonly, copy) NSArray<NSOperation *> *dependencies;`` 在当前操作开始执行之前完成执行的所有操作对象数组。



### NSOperationQueue 常用属性和方法

1. 取消/暂停/恢复操作
	* ``- (void)cancelAllOperations;`` 可以取消队列的所有操作。
	* ``- (BOOL)isSuspended;`` 判断队列是否处于暂停状态。 YES 为暂停状态，NO 为恢复状态。
	* ``- (void)setSuspended:(BOOL)b;`` 可设置操作的暂停和恢复，YES 代表暂停队列，NO 代表恢复队列。
2. 操作同步
	* ``- (void)waitUntilAllOperationsAreFinished;`` 阻塞当前线程，直到队列中的操作全部执行完毕。
3. 添加/获取操作`
	* ``- (void)addOperationWithBlock:(void (^)(void))block;`` 向队列中添加一个 NSBlockOperation 类型操作对象。
	* ``- (void)addOperations:(NSArray *)ops waitUntilFinished:(BOOL)wait; ``向队列中添加操作数组，wait 标志是否阻塞当前线程直到所有操作结束
	* ``- (NSArray *)operations;`` 当前在队列中的操作数组（某个操作执行结束后会自动从这个数组清除）。
	* ``- (NSUInteger)operationCount;`` 当前队列中的操作数。
4. 获取队列
	* ``+ (id)currentQueue;`` 获取当前队列，如果当前线程不是在 NSOperationQueue 上运行则返回 nil。
	* ``+ (id)mainQueue;`` 获取主队列。













