# 5.2 pthread

> pthread是POSIX thread的简写，一套通用的多线程API，适用于Unix、Linux、Windows等系统，跨平台、可移植，使用难度大，C语言框架，线程生命周期由程序员管理，由于iOS开发几乎用不到，以下就简单运用pthread开启一个子线程，用来处理耗时操作
> 
> 


## pthread 使用方法

1. 首先要包含头文件#import <pthread.h>

2. 其次要创建线程，并开启线程执行任务

	```
	// 1. 创建线程: 定义一个pthread_t类型变量
	pthread_t thread;
	// 2. 开启线程: 执行任务
	pthread_create(&thread, NULL, run, NULL);
	// 3. 设置子线程的状态设置为 detached，该线程运行结束后会自动释放所有资源
	pthread_detach(thread);
	
	void * run(void *param)    // 新线程调用方法，里边为需要执行的任务
	{
	    NSLog(@"%@", [NSThread currentThread]);
	
	    return NULL;
	}
	
	```

3. ``pthread_create(&thread, NULL, run, NULL); ``中各项参数含义：

	* 第一个参数&thread是线程对象，指向线程标识符的指针
	* 第二个是线程属性，可赋值NULL
	* 第三个run表示指向函数的指针(run对应函数里是需要在新线程中执行的任务)
	* 第四个是运行函数的参数，可赋值NULL


###  pthread 其他相关方法

* pthread_create() 创建一个线程
* pthread_exit() 终止当前线程
* pthread_cancel() 中断另外一个线程的运行
* pthread_join() 阻塞当前的线程，直到另外一个线程运行结束
* pthread_attr_init() 初始化线程的属性
* pthread_attr_setdetachstate() 设置脱离状态的属性（决定这个线程在终止时是否可以被结合）
* pthread_attr_getdetachstate() 获取脱离状态的属性
* pthread_attr_destroy() 删除线程的属性
* pthread_kill() 向线程发送一个信号






```

// 创建线程，并且在线程中执行 demo 函数
- (void)pthreadDemo {
     返回值：
     - 若线程创建成功，则返回0
     - 若线程创建失败，则返回出错编号
     */
    pthread_t threadId = NULL;
    NSString *str = @"Hello Pthread";
    // 这边的demo函数名作为第三个参数写在这里可以在其前面加一个&，也可以不加，因为函数名就代表了函数的地址。
    int result = pthread_create(&threadId, NULL, demo, (__bridge void *)(str));

    if (result == 0) {
        NSLog(@"创建线程 OK");
    } else {
        NSLog(@"创建线程失败 %d", result);
    }
    // pthread_detach:设置子线程的状态设置为detached,则该线程运行结束后会自动释放所有资源。
    pthread_detach(threadId);
}

// 后台线程调用函数
void *demo(void *params) {
    NSString *str = (__bridge NSString *)(params);

    NSLog(@"%@ - %@", [NSThread currentThread], str);
    return NULL;
}


```





