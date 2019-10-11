# 16.2 Method Swizzling利弊

>使用 Method Swizzling 编程就好比切菜时使用锋利的刀，一些人因为担心切到自己所以害怕锋利的刀具，可是事实上，使用钝刀往往更容易出事，而利刀更为安全。

>Method swizzling 可以帮助我们写出更好的，更高效的，易维护的代码。但是如果滥用它，也将会导致难以排查的bug。
>




## Method Swizzling的陷阱

### Method swizzling is not atomic


* 我所见过的使用method swizzling实现的方法在并发使用时基本都是安全的。95%的情况里这都不会是个问题。通常你替换一个方法的实现，是希望它在整个程序的生命周期里有效的。

* 也就是说，你会把 method swizzling 修改方法实现的操作放在一个加号方法 +(void)load里，并在应用程序的一开始就调用执行。你将不会碰到并发问题。假如你在 +(void)initialize初始化方法中进行swizzle，那么……rumtime可能死于一个诡异的状态。


### Changes behavior of un-owned code

* 这是swizzling的一个问题。我们的目标是改变某些代码。swizzling方法是一件灰常灰常重要的事，当你不只是对一个NSButton类的实例进行了修改，而是程序中所有的NSButton实例。因此在swizzling时应该多加小心，但也不用总是去刻意避免。

* 想象一下，如果你重写了一个类的方法，而且没有调用父类的这个方法，这可能会引起问题。大多数情况下，父类方法期望会被调用（至少文档是这样说的）。如果你在swizzling实现中也这样做了，这会避免大部分问题。还是调用原始实现吧，如若不然，你会费很大力气去考虑代码的安全问题。



### Possible naming conflicts

* 命名冲突贯穿整个Cocoa的问题. 我们常常在类名和类别方法名前加上前缀。不幸的是，命名冲突仍是个折磨。但是swizzling其实也不必过多考虑这个问题。我们只需要在原始方法命名前做小小的改动来命名就好，比如通常我们这样命名：

```
@interface NSView : NSObject  
- (void)setFrame:(NSRect)frame;  
@end  
  
  
@implementation NSView (MyViewAdditions)  
  
  
- (void)my_setFrame:(NSRect)frame {  
    // do custom work  
    [self my_setFrame:frame];  
}  
  
  
+ (void)load {  
    [self swizzle:@selector(setFrame:) with:@selector(my_setFrame:)];  
}  
  
  
@end  

```

这段代码运行正确，但是如果my_setFrame: 在别处被定义了会发生什么呢？

这个问题不仅仅存在于swizzling，这里有一个替代的变通方法：

```
@implementation NSView (MyViewAdditions)  
  
  
static void MySetFrame(id self, SEL _cmd, NSRect frame);  
static void (*SetFrameIMP)(id self, SEL _cmd, NSRect frame);  
  
  
static void MySetFrame(id self, SEL _cmd, NSRect frame) {  
    // do custom work  
    SetFrameIMP(self, _cmd, frame);  
}  
  
  
+ (void)load {  
    [self swizzle:@selector(setFrame:) with:(IMP)MySetFrame store:(IMP *)&SetFrameIMP];  
}  
  
  
@end  

```



* 看起来不那么Objectice-C了（用了函数指针），这样避免了selector的命名冲突

```
typedef IMP *IMPPointer;  
  
  
BOOL class_swizzleMethodAndStore(Class class, SEL original, IMP replacement, IMPPointer store) {  
    IMP imp = NULL;  
    Method method = class_getInstanceMethod(class, original);  
    if (method) {  
        const char *type = method_getTypeEncoding(method);  
        imp = class_replaceMethod(class, original, replacement, type);  
        if (!imp) {  
            imp = method_getImplementation(method);  
        }  
    }  
    if (imp && store) { *store = imp; }  
    return (imp != NULL);  
}  
  
  
@implementation NSObject (FRRuntimeAdditions)  
+ (BOOL)swizzle:(SEL)original with:(IMP)replacement store:(IMPPointer)store {  
    return class_swizzleMethodAndStore(self, original, replacement, store);  
}  
@end  

```


### Swizzling changes the method's arguments

想正常调用method swizzling 将会是个问题。

```
[self my_setFrame:frame];  

```

直接调用my_setFrame: ， runtime做的是

```
objc_msgSend(self, @selector(my_setFrame:), frame);  

```

* untime去寻找my_setFrame:的方法实现, _cmd参数为 my_setFrame: ，但是事实上runtime找到的方法实现是原始的 setFrame: 的。

* 一个简单的解决办法：使用上面介绍的swizzling定义。


### The order of swizzles matters


多个swizzle方法的执行顺序也需要注意。假设 setFrame: 只定义在NSView中，想像一下按照下面的顺序执行：

```
[NSButton swizzle:@selector(setFrame:) with:@selector(my_buttonSetFrame:)];  
[NSControl swizzle:@selector(setFrame:) with:@selector(my_controlSetFrame:)];  
[NSView swizzle:@selector(setFrame:) with:@selector(my_viewSetFrame:)];  

```

> What happens when the method on NSButton is swizzled? Well most swizzling will ensure that it's not replacing the implementation of setFrame: for all views, so it will pull up the instance method. This will use the existing implementation to re-define setFrame: in the NSButton class so that exchanging implementations doesn't affect all views. The existing implementation is the one defined on NSView. The same thing will happen when swizzling on NSControl (again using the NSView implementation).

>When you call setFrame: on a button, it will therefore call your swizzled method, and then jump straight to the setFrame: method originally defined on NSView. The NSControl and NSView swizzled implementations will not be called.

But what if the order were:

```
[NSView swizzle:@selector(setFrame:) with:@selector(my_viewSetFrame:)];  
[NSControl swizzle:@selector(setFrame:) with:@selector(my_controlSetFrame:)];  
[NSButton swizzle:@selector(setFrame:) with:@selector(my_buttonSetFrame:)];  

```

>Since the view swizzling takes place first, the control swizzling will be able to pull up the right method. Likewise, since the control swizzling was before the button swizzling, the button will pull up the control's swizzled implementation of setFrame:. This is a bit confusing, but this is the correct order. How can we ensure this order of things?

>Again, just use load to swizzle things. If you swizzle in load and you only make changes to the class being loaded, you'll be safe. The load method guarantees that the super class load method will be called before any subclasses. We'll get the exact right order!


总结一下就是：多个有继承关系的类的对象swizzle时，先从父对象开始。 这样才能保证子类方法拿到父类中的被swizzle的实现。在+(void)load中swizzle不会出错，就是因为load类方法会默认从父类开始调用。


### Difficult to understand (looks recursive)

看起来像递归，但是看看上面已经给出的 swizzling 封装方法, 使用起来就很易读懂.
这个问题是已完全解决的了！


### Difficult to debug

debug时打出的backtrace，其中掺杂着被swizzle的方法名，一团糟啊！上面介绍的swizzle方案，使backtrace中打印出的方法名还是很清晰的。但仍然很难去debug，因为很难记住swizzling影响过什么。给你的代码写好文档（即使只有你一个人会看到）。养成一个好习惯，不会比调试多线程问题还难的。



###  官方API版本兼容问题

* 每次iOS版本的更新都会出现一些新的API，这些API在之前的版本中就不存在，而你在交换一个方法时如果不考虑它是否存在，那就会导致严重的错误

```

- (void)openURL:(NSURL*)url options:(NSDictionary<NSString *, id> *)options completionHandler:(void (^ __nullable)(BOOL success))completion

```

这个方法只在iOS10之后有，别人在使用的时候也会先判断这个方法是否会被响应，但是我们看下面的交换代码会先添加这个方法，如果这个方法不存在，那么本来不会响应就变成了会响应，那么在iOS10之前的系统就会进入这个方法，导致死循环。


```
Class class = [self class];

    Method method1 = class_getInstanceMethod(class, sel1);
    Method method2 = class_getInstanceMethod(class, sel2);

    BOOL didAddMethod =
    class_addMethod(class,
                    sel1,
                    method_getImplementation(method2),
                    method_getTypeEncoding(method2));

    if (didAddMethod) {
        class_replaceMethod(class,
                            sel2,
                            method_getImplementation(method1),
                            method_getTypeEncoding(method1));
    } else {
        method_exchangeImplementations(method1, method2);
    }

```

解决这个问题也很简单就是交换前做一个判断：

```
if (![class instancesRespondToSelector:sel1] || ![class instancesRespondToSelector:sel2]) {
        return ;
    }

```

我们可以将其封装，将其放在NSObject的Category中

```

#import <Foundation/Foundation.h>

@interface NSObject (DPExtension)

+ (void)intanceMethodExchangeWithOriginSelector:(SEL)sel1 swizzledSelector:(SEL)sel2;

@end


#import "NSObject+DPExtension.h"
#import <objc/runtime.h>

@implementation NSObject (DPExtension)

+ (void)intanceMethodExchangeWithOriginSelector:(SEL)sel1 swizzledSelector:(SEL)sel2 {

    Class class = [self class];

    if (![class instancesRespondToSelector:sel1] || ![class instancesRespondToSelector:sel2]) {
        return ;
    }

    Method method1 = class_getInstanceMethod(class, sel1);
    Method method2 = class_getInstanceMethod(class, sel2);

    BOOL didAddMethod =
    class_addMethod(class,
                    sel1,
                    method_getImplementation(method2),
                    method_getTypeEncoding(method2));

    if (didAddMethod) {
        class_replaceMethod(class,
                            sel2,
                            method_getImplementation(method1),
                            method_getTypeEncoding(method1));
    } else {
        method_exchangeImplementations(method1, method2);
    }
}

@end


```

[hook调用顺序](http://yulingtianxia.com/blog/2017/04/17/Objective-C-Method-Swizzling/)
