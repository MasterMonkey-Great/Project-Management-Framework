# 第十四章 RunTime


### Objective-C 扩展了 C 语言，并加入了面向对象特性和 Smalltalk 式的消息传递机制。而这个扩展的核心是一个用 C 和 编译语言 写的 Runtime 库。它是 Objective-C 面向对象和动态机制的基石。

* RunTime简称运行时。OC就是运行时机制，其中最主要的是消息机制。对于C语言，函数的调用在编译的时候会决定调用哪个函数。对于OC的函数，属于动态调用过程，在编译的时候并不能决定真正调用哪个函数，只有在真正运行的时候才会根据函数的名称找到对应的函数来调用。

### Objective-C 是一个动态语言，这意味着它不仅需要一个编译器，也需要一个运行时系统来动态得创建类和对象、进行消息传递和转发。理解 Objective-C 的 Runtime 机制可以帮我们更好的了解这个语言，适当的时候还能对语言进行扩展，从系统层面解决项目中的一些设计或技术问题。了解 Runtime ，要先了解它的核心 - 消息传递 （Messaging）。



根据以上的函数的前缀 可以大致了解到层级关系。

对于以objc_开头的方法，则是runtime最终的管家，可以获取内存中类的加载信息,类的列表，关联对象和关联属性等操作。

例如：使用runtime对当前的应用中加载的类进行打印，别被吓一跳。

```
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event {
    unsigned int count = 0;
    Class *classes = objc_copyClassList(&count);
    for (int i = 0; i < count; i++) {
        const char *cname = class_getName(classes[i]);
        printf("%s\n", cname);
    }
}

```








[探寻Runtime本质](https://www.jianshu.com/p/57e3f555756e)
