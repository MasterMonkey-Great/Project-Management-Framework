# 5.7 锁


　　日常开发过程中，为了提升程序运行效率，以及用户体验，我们经常使用多线程。在使用多线程的过程中，难免会遇到资源竞争问题。我们采用锁的机制来确保线程安全。


　　当一个线程访问数据的时候，其他的线程不能对其进行访问，直到该线程访问完毕。即，同一时刻，对同一个数据操作的线程只有一个。只有确保了这样，才能使数据不会被其他线程污染。而线程不安全，则是在同一时刻可以有多个线程对该数据进行访问，从而得不到预期的结果。

   比如写文件和读文件，当一个线程在写文件的时候，理论上来说，如果这个时候另一个线程来直接读取的话，那么得到将是不可预期的结果。

为了线程安全，我们可以使用锁的机制来确保，同一时刻只有同一个线程来对同一个数据源进行访问。



## 互斥锁

一种用来防止多个线程同一时刻对共享资源进行访问的信号量，它的原子性确保了如果一个线程锁定了一个互斥量，将没有其他线程在同一时间可以锁定这个互斥量。它的唯一性确保了只有它解锁了这个互斥量，其他线程才可以对其进行锁定。当一个线程锁定一个资源的时候，其他对该资源进行访问的线程将会被挂起，直到该线程解锁了互斥量，其他线程才会被唤醒，进一步才能锁定该资源进行操作。

####  @synchronized：对象级别所，互斥锁，性能较差不推荐使用

```
    @synchronized(这里添加一个OC对象，一般使用self) {

        这里写要加锁的代码

    }
```

* @synchronized使用注意点

　　1. 加锁的代码尽量少

　　2. 添加的OC对象必须在多个线程中都是同一对象，下面举一个反例 

```
- (void)viewDidLoad {
    [super viewDidLoad];
    //设置票的数量为5
    _tickets = 5;
    //线程一
    NSThread *threadOne = [[NSThread alloc] initWithTarget:self selector:@selector(saleTickets) object:nil];
    threadOne.name = @"threadOne";
    //线程二
    NSThread *threadTwo = [[NSThread alloc] initWithTarget:self selector:@selector(saleTickets) object:nil];
     
    //开启线程
    [threadOne start];
    [threadTwo start];
}
- (void)saleTickets
{
    NSObject *object = [[NSObject alloc] init];
    while (1)
    {
        @synchronized(object) {
            [NSThread sleepForTimeInterval:1];
            if (_tickets > 0)
            {
                _tickets--;
                NSLog(@"剩余票数= %ld",_tickets);
            }
            else
            {
                NSLog(@"票卖完了");
                break;
            }
        }
    }   
}	
	
```

* 结果卖票又出错了，出现这个原因的问题是每个线程都会创建一个object对象，锁后面加的object在不同线程中就不同了；

* 把@synchronized（object）改成 @synchronized（self）就能得到了正确结果



#### NSLock

> NSLock实现了最基本的互斥锁，遵循了 NSLocking 协议，通过 lock 和 unlock 来进行锁定和解锁。其使用也非常简单
> 


```
- (void)doSomething {
   [self.lock lock];
   //TODO: do your stuff
   [self.lock unlock];
}

```

由于是互斥锁，当一个线程进行访问的时候，该线程获得锁，其他线程进行访问的时候，将被操作系统挂起，直到该线程释放锁，其他线程才能对其进行访问，从而却确保了线程安全。但是如果连续锁定两次，则会造成死锁问题。

```
@interface ViewController ()
{
    NSLock *mutexLock;
}
 
@property (assign, nonatomic)NSInteger tickets;
@end
 
@implementation ViewController
 
- (void)viewDidLoad {
    [super viewDidLoad];
    //创建锁
    mutexLock = [[NSLock alloc] init];
     
    //设置票的数量为5
    _tickets = 5;
    //线程一
    NSThread *threadOne = [[NSThread alloc] initWithTarget:self selector:@selector(saleTickets) object:nil];
    threadOne.name = @"threadOne";
    //线程二
    NSThread *threadTwo = [[NSThread alloc] initWithTarget:self selector:@selector(saleTickets) object:nil];
     
    //开启线程
    [threadOne start];
    [threadTwo start]; 
}
- (void)saleTickets
{
    while (1)
    {
        [NSThread sleepForTimeInterval:1];
        //加锁
        [mutexLock lock];
        if (_tickets > 0)
        {
            _tickets--;
            NSLog(@"剩余票数= %ld",_tickets);
        }
        else
        {
            NSLog(@"票卖完了");
            break;
        }
        //解锁
        [mutexLock unlock];   
    }
}

```
* 修改代码. NSLock: 使用注意，不能多次调用 lock方法,会造成死锁

```
- (void)saleTickets
{
    while (1)
    {
        [NSThread sleepForTimeInterval:1];
        //加锁
        [mutexLock lock];
        
        if (_tickets > 0)
        {
            _tickets--;
            NSLog(@"剩余票数= %ld",_tickets);
        }
        else
        {
            NSLog(@"票卖完了");
            break;
        }
        
        if (_tickets == 2){
            [mutexLock lock];
        }
        //解锁
        [mutexLock unlock];
    }
}


```

#### ``pthread_mutex`` 


POSIX 互斥锁是一种超级易用的互斥锁，使用的时候，只需要初始化一个 pthread_mutex_t 用 pthread_mutex_lock 来锁定 pthread_mutex_unlock 来解锁，当使用完成后，记得调用 pthread_mutex_destroy 来销毁锁。

```
    pthread_mutex_init(&lock,NULL);
    pthread_mutex_lock(&lock);
    //do your stuff
    pthread_mutex_unlock(&lock);
    pthread_mutex_destroy(&lock);

```

### pthread_rwlock

读写锁，在对文件进行操作的时候，写操作是排他的，一旦有多个线程对同一个文件进行写操作，后果不可估量，但读是可以的，多个线程读取时没有问题的。

* 当读写锁被一个线程以读模式占用的时候，写操作的其他线程会被阻塞，读操作的其他线程还可以继续进行。
* 当读写锁被一个线程以写模式占用的时候，写操作的其他线程会被阻塞，读操作的其他线程也被阻塞。

```
// 初始化
pthread_rwlock_t rwlock = PTHREAD_RWLOCK_INITIALIZER
// 读模式
pthread_rwlock_wrlock(&lock);
// 写模式
pthread_rwlock_rdlock(&lock);
// 读模式或者写模式的解锁
pthread_rwlock_unlock(&lock);


```

```
 dispatch_async(dispatch_get_global_queue(0, 0), ^{
        [self readBookWithTag:1];
    });
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        [self readBookWithTag:2];
    });
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        [self writeBook:3];
    });
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        [self writeBook:4];
    });
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        [self readBookWithTag:5];
    });
- (void)readBookWithTag:(NSInteger )tag {
    pthread_rwlock_rdlock(&rwLock);
    NSLog(@"start read ---- %ld",tag);
    self.path = [[NSBundle mainBundle] pathForResource:@"1" ofType:@".doc"];
    self.contentString = [NSString stringWithContentsOfFile:self.path encoding:NSUTF8StringEncoding error:nil];
    NSLog(@"end   read ---- %ld",tag);
    pthread_rwlock_unlock(&rwLock);
}
- (void)writeBook:(NSInteger)tag {
    pthread_rwlock_wrlock(&rwLock);
    NSLog(@"start wirte ---- %ld",tag);
    [self.contentString writeToFile:self.path atomically:YES encoding:NSUTF8StringEncoding error:nil];
    NSLog(@"end   wirte ---- %ld",tag);
    pthread_rwlock_unlock(&rwLock);
}
start   read ---- 1
start   read ---- 2
end    read ---- 1
end    read ---- 2
start   wirte ---- 3
end    wirte ---- 3
start   wirte ---- 4
end    wirte ---- 4
start   read ---- 5
end    read ---- 5


```






## 递归锁

### NSRecursiveLock

递归锁，顾名思义，可以被一个线程多次获得，而不会引起死锁。它记录了成功获得锁的次数，每一次成功的获得锁，必须有一个配套的释放锁和其对应，这样才不会引起死锁。只有当所有的锁被释放之后，其他线程才可以获得锁

```
NSRecursiveLock *theLock = [[NSRecursiveLock alloc] init];
void MyRecursiveFunction(int value)
{
    [theLock lock];
    if (value != 0)
    {
        --value;
        MyRecursiveFunction(value);
    }
    [theLock unlock];
}
MyRecursiveFunction(5);

```

```
@interface ViewController ()
{
    NSRecursiveLock *rsLock;
}
 
@property (assign, nonatomic)NSInteger tickets;
 
 
@end
 
@implementation ViewController
 
- (void)viewDidLoad {
    [super viewDidLoad];
    //创建锁递归锁
    rsLock = [[NSRecursiveLock alloc] init];   
    //设置票的数量为5
    _tickets = 5;
    //线程一
    NSThread *threadOne = [[NSThread alloc] initWithTarget:self selector:@selector(saleTickets) object:nil];
    threadOne.name = @"threadOne";
    //线程二
    NSThread *threadTwo = [[NSThread alloc] initWithTarget:self selector:@selector(saleTickets) object:nil];
    //开启线程
    [threadOne start];
    [threadTwo start];   
}
- (void)saleTickets
{
    while (1)
    {
        [NSThread sleepForTimeInterval:1];
        //加锁,递归锁可以多次加锁
        [rsLock lock];
        [rsLock lock];
        if (_tickets > 0)
        {
            _tickets--;
            NSLog(@"剩余票数= %ld",_tickets);
        }
        else
        {
            NSLog(@"票卖完了");
            break;
        }
        //解锁，只有对应次数解锁，其他线程才能访问。
        [rsLock unlock];
        [rsLock unlock];  
    }   
}

```


## 自旋锁

#### atomic 原子属性(线程安全)

自旋锁，当新线程访问代码时，如果发现有其他线程正在锁定代码，新线程会用死循环的方式，一直等待锁定的代码执行完成。相当于不停尝试执行代码，比较消耗性能。

属性修饰atomic本身就有一把自旋锁。

下面说一下属性修饰nonatomic 和 atomic

```
nonatomic 非原子属性,同一时间可以有很多线程读和写
atomic 原子属性(线程安全)，保证同一时间只有一个线程能够写入(但是同一个时间多个线程都可以取值)，atomic 本身就有一把锁(自旋锁)
atomic：线程安全，需要消耗大量的资源
nonatomic：非线程安全，不过效率更高，一般使用nonatomic

```



#### OSSpinLock

自旋锁，和互斥锁类似，都是为了保证线程安全的锁。但二者的区别是不一样的，对于互斥锁，当一个线程获得这个锁之后，其他想要获得此锁的线程将会被阻塞，直到该锁被释放。但自选锁不一样，当一个线程获得锁之后，其他线程将会一直循环在哪里查看是否该锁被释放。所以，此锁比较适用于锁的持有者保存时间较短的情况下。

```
// 初始化
spinLock = OS_SPINLOCK_INIT;
// 加锁
OSSpinLockLock(&spinLock);
// 解锁
OSSpinLockUnlock(&spinLock);

```


然而，YYKit 作者 @ibireme 的文章也有说这个自旋锁存在优先级反转问题，具体文章可以戳 [不再安全的 OSSpinLock](https://blog.ibireme.com/2016/01/16/spinlock_is_unsafe_in_ios/?utm_source=tuicool&utm_medium=referral)。


#### ``os_unfair_lock``


自旋锁已经不在安全，然后苹果又整出来个 os_unfair_lock_t (╯‵□′)╯︵┻━┻

这个锁解决了优先级反转问题。

```
    os_unfair_lock_t unfairLock;
    unfairLock = &(OS_UNFAIR_LOCK_INIT);
    os_unfair_lock_lock(unfairLock);
    os_unfair_lock_unlock(unfairLock);

```



## 信号量机制实现锁

信号量机制实现锁，等待信号，和发送信号，正如前边所说的看门人一样，当有多个线程进行访问的时候，只要有一个获得了信号，其他线程的就必须等待该信号释放。

```
- (void)semphone:(NSInteger)tag {
    dispatch_semaphore_wait(semaphore, DISPATCH_TIME_NOW);
    // do your stuff
    dispatch_semaphore_signal(semaphore);
}

```



##  条件锁

#### NSCondition:可以理解为互斥锁和条件锁的结合

用生产者消费者中的例子可以很好的理解NSCondition

1. 生产者要取得锁，然后去生产，生产后将生产的商品放入库房，如果库房满了，则wait，就释放锁，直到其它线程唤醒它去生产，如果没有满，则生产商品后调用signal，可以唤醒在此condition上等待的线程。

2. 消费者要取得锁，然后去消费，如果当前没有商品，则wait,释放锁，直到有线程去唤醒它消费，如果有商品，则消费后会通知正在等待的生产者去生产商品。

3. 生产者和消费者的关键是：当库房已满时，生产者等待，不再继续生产商品，当库房已空时，消费者等待，不再继续消费商品，走到库房有商品时，会由生产者通知消费来消费。

```

- (void)conditionTest
{
    //创建数组存放商品
    products = [[NSMutableArray alloc] init];
    condition = [[NSCondition alloc] init];
     
    [NSThread detachNewThreadSelector:@selector(createProducter) toTarget:self withObject:nil];
    [NSThread detachNewThreadSelector:@selector(createConsumenr) toTarget:self withObject:nil];
}
 
- (void)createConsumenr
{
    while (1) {
        //模拟消费商品时间,让它比生产慢一点
        [NSThread sleepForTimeInterval:arc4random()%10 * 0.1 + 1.5];
        [condition lock];
        while (products.count == 0) {
            NSLog(@"商品为0，等待生产");
            [condition wait];
        }
        [products removeLastObject];
        NSLog(@"消费了一个商品,商品数 = %ld",products.count);
        [condition signal];
        [condition unlock];
    }
     
}
 
- (void)createProducter
{
    while (1) {
        //模拟生产商品时间
        [NSThread sleepForTimeInterval:arc4random()%10 * 0.1 + 0.5];
        [condition lock];
        while (products.count == 5)
        {
            NSLog(@"商品满了，等待消费");
            [condition wait];
        }
        [products addObject:[[NSObject alloc] init]];
        NSLog(@"生产了一个商品,商品数%ld",products.count);
        [condition signal];
        [condition unlock];
    }
     
}

```


#### NSConditionLock

NSConditionLock 对象所定义的互斥锁可以在使得在某个条件下进行锁定和解锁。它和 NSCondition 很像，但实现方式是不同的。

当两个线程需要特定顺序执行的时候，例如生产者消费者模型，则可以使用 NSConditionLock 。当生产者执行执行的时候，消费者可以通过特定的条件获得锁，当生产者完成执行的时候，它将解锁该锁，然后把锁的条件设置成唤醒消费者线程的条件。锁定和解锁的调用可以随意组合，lock 和 unlockWithCondition: 配合使用 lockWhenCondition: 和 unlock 配合使用。

```
- (void)producer {
    while (YES) {
        [self.conditionLock lock];
        NSLog(@"have something");
        self.count++;
        [self.conditionLock unlockWithCondition:1];
    }
}
- (void)consumer {
    while (YES) {
        [self.conditionLock lockWhenCondition:1];
        NSLog(@"use something");
        self.count--;
        [self.conditionLock unlockWithCondition:0];
    }
}

```


当生产者释放锁的时候，把条件设置成了1。这样消费者可以获得该锁，进而执行程序，如果消费者获得锁的条件和生产者释放锁时给定的条件不一致，则消费者永远无法获得锁，也不能执行程序。同样，如果消费者释放锁给定的条件和生产者获得锁给定的条件不一致的话，则生产者也无法获得锁，程序也不能执行。



#### POSIX Conditions


POSIX 条件锁需要互斥锁和条件两项来实现，虽然看起来没什么关系，但在运行时中，互斥锁将会与条件结合起来。线程将被一个互斥和条件结合的信号来唤醒。

首先初始化条件和互斥锁，当 ready_to_go 为 flase 的时候，进入循环，然后线程将会被挂起，直到另一个线程将 ready_to_go 设置为 true 的时候，并且发送信号的时候，该线程会才被唤醒。

```
pthread_mutex_t mutex;
pthread_cond_t condition;
Boolean     ready_to_go = true;
void MyCondInitFunction()
{
    pthread_mutex_init(&mutex);
    pthread_cond_init(&condition, NULL);
}
void MyWaitOnConditionFunction()
{
    // Lock the mutex.
    pthread_mutex_lock(&mutex);
    // If the predicate is already set, then the while loop is bypassed;
    // otherwise, the thread sleeps until the predicate is set.
    while(ready_to_go == false)
    {
        pthread_cond_wait(&condition, &mutex);
    }
    // Do work. (The mutex should stay locked.)
    // Reset the predicate and release the mutex.
    ready_to_go = false;
    pthread_mutex_unlock(&mutex);
}
void SignalThreadUsingCondition()
{
    // At this point, there should be work for the other thread to do.
    pthread_mutex_lock(&mutex);
    ready_to_go = true;
    // Signal the other thread to begin work.
    pthread_cond_signal(&condition);
    pthread_mutex_unlock(&mutex);
}

```









































































应当针对不同的操作使用不同的锁，而不能一概而论那种锁的加锁解锁速度快。

* 当进行文件读写的时候，使用 pthread_rwlock 较好，文件读写通常会消耗大量资源，而使用互斥锁同时读文件的时候会阻塞其他读文件线程，而 pthread_rwlock 不会。

* 当性能要求较高时候，可以使用 pthread_mutex 或者 dispath_semaphore，由于 OSSpinLock 不能很好的保证线程安全，而在只有在 iOS10 中才有 os_unfair_lock ，所以，前两个是比较好的选择。既可以保证速度，又可以保证线程安全。


* 对于 NSLock 及其子类，速度来说 NSLock < NSCondition < NSRecursiveLock < NSConditionLock 。







## 死锁

　　概念：死锁是指两个或两个以上的进程（线程）在执行过程中，由于竞争资源或者由于彼此通信而造成的一种阻塞的现象，若无外力作用，它们都将无法推进下去。

产生死锁的4个必要条件

 

1. 互斥条件：指进程对所分配到的资源进行排它性使用，即在一段时间内某资源只由一个进程占用。如果此时还有其它进程请求资源，则请求者只能等待，直至占有资源的进程用毕释放。
 

2. 请求和保持条件：指进程已经保持至少一个资源，但又提出了新的资源请求，而该资源已被其它进程占有，此时请求进程阻塞，但又对自己已获得的其它资源保持不放。
 

3. 不剥夺条件：指进程已获得的资源，在未使用完之前，不能被剥夺，只能在使用完时由自己释放。
 

4. 环路等待条件：指在发生死锁时，必然存在一个进程——资源的环形链，即进程集合{P0，P1，P2，···，Pn}中的P0正在等待一个P1占用的资源；P1正在等待P2占用的资源，……，Pn正在等待已被P0占用的资源。





[Linux C互斥锁和条件变量（POSIX标准）](Linux C互斥锁和条件变量)







































