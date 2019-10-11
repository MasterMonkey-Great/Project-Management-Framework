# 14.4 Runtime应用


Runtime简直就是做大型框架的利器。它的应用场景非常多，下面就介绍一些常见的应用场景。

* 关联对象(Objective-C Associated Objects)给分类增加属性
* 方法魔法(Method Swizzling)方法添加和替换和KVO实现
* 消息转发(热更新)解决Bug(JSPatch)
* 实现NSCoding的自动归档和自动解档
* 实现字典和模型的自动转换(MJExtension)



### 关联对象(Objective-C Associated Objects)给分类增加属性

我们都是知道分类是不能自定义属性和变量的。下面通过关联对象实现给分类添加属性。


关联对象Runtime提供了下面几个接口：

```
//关联对象
void objc_setAssociatedObject(id object, const void *key, id value, objc_AssociationPolicy policy)
//获取关联的对象
id objc_getAssociatedObject(id object, const void *key)
//移除关联的对象
void objc_removeAssociatedObjects(id object)

```

参数解释:

* id object：被关联的对象
* const void *key：关联的key，要求唯一
* id value：关联的对象
* objc_AssociationPolicy policy：内存管理的策略


内存管理的策略：

```
typedef OBJC_ENUM(uintptr_t, objc_AssociationPolicy) {
    OBJC_ASSOCIATION_ASSIGN = 0,           /**< Specifies a weak reference to the associated object. */
    OBJC_ASSOCIATION_RETAIN_NONATOMIC = 1, /**< Specifies a strong reference to the associated object. 
                                            *   The association is not made atomically. */
    OBJC_ASSOCIATION_COPY_NONATOMIC = 3,   /**< Specifies that the associated object is copied. 
                                            *   The association is not made atomically. */
    OBJC_ASSOCIATION_RETAIN = 01401,       /**< Specifies a strong reference to the associated object.
                                            *   The association is made atomically. */
    OBJC_ASSOCIATION_COPY = 01403          /**< Specifies that the associated object is copied.
                                            *   The association is made atomically. */
};


```

下面实现一个``UIView``的``Category``添加自定义属性``defaultColor``。

```
#import "ViewController.h"
#import "objc/runtime.h"

@interface UIView (DefaultColor)

@property (nonatomic, strong) UIColor *defaultColor;

@end

@implementation UIView (DefaultColor)

@dynamic defaultColor;

static char kDefaultColorKey;

- (void)setDefaultColor:(UIColor *)defaultColor {
    objc_setAssociatedObject(self, &kDefaultColorKey, defaultColor, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}

- (id)defaultColor {
    return objc_getAssociatedObject(self, &kDefaultColorKey);
}

@end

@interface ViewController ()

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    
    UIView *test = [UIView new];
    test.defaultColor = [UIColor blackColor];
    NSLog(@"%@", test.defaultColor);
}

@end

```


通过关联对象实现的属性的内存管理也是有ARC管理的，所以我们只需要给定适当的内存策略就行了，不需要操心对象的释放。

内存管理的策略 属性修饰 

* ``OBJC_ASSOCIATION_ASSIGN``  -->  ``@property (assign) 或 @property (unsafe_unretained)``  -->  指定一个关联对象的弱引用。

* ``OBJC_ASSOCIATION_RETAIN_NONATOMIC``  -->   `` @property (nonatomic, strong) ``   -->  @property (nonatomic, strong) 指定一个关联对象的强引用，不能被原子化使用。

* `` OBJC_ASSOCIATION_COPY_NONATOMIC ``   -->   ``@property (nonatomic, copy)``    -->   指定一个关联对象的copy引用，不能被原子化使用。

* ``OBJC_ASSOCIATION_RETAIN``   -->   `` @property (atomic, strong) ``  -->    指定一个关联对象的强引用，能被原子化使用。

* ``OBJC_ASSOCIATION_COPY ``   --> `` @property (atomic, copy) `` -->  指定一个关联对象的copy引用，能被原子化使用。



### 方法魔法(Method Swizzling)方法添加和替换和KVO实现

##### 方法添加

```
//class_addMethod(Class  _Nullable __unsafe_unretained cls, SEL  _Nonnull name, IMP  _Nonnull imp, const char * _Nullable types)
class_addMethod([self class], sel, (IMP)fooMethod, "v@:");


```

* cls 被添加方法的类
* name 添加的方法的名称的SEL
* imp 方法的实现。该函数必须至少要有两个参数，self,_cmd
* 类型编码


### KVO实现

> 全称是Key-value observing，翻译成键值观察。提供了一种当其它对象属性被修改的时候能通知当前对象的机制。再MVC大行其道的Cocoa中，KVO机制很适合实现model和controller类之间的通讯。
> 
> 

``KVO``的实现依赖于 ``Objective-C`` 强大的 ``Runtime``，当观察某对象 ``A`` 时，``KVO`` 机制动态创建一个对象A当前类的子类，并为这个新的子类重写了被观察属性 ``keyPath`` 的 ``setter`` 方法。``setter`` 方法随后负责通知观察对象属性的改变状况。

Apple 使用了 ``isa-swizzling`` 来实现 KVO 。当观察对象A时，KVO机制动态创建一个新的名为：``NSKVONotifying_A``的新类，该类继承自对象A的本类，且 KVO 为 ``NSKVONotifying_A`` 重写观察属性的 ``setter`` 方法，``setter`` 方法会负责在调用原 ``setter`` 方法之前和之后，通知所有观察对象属性值的更改情况。

NSKVONotifying_A 类剖析

```
NSLog(@"self->isa:%@",self->isa);  
NSLog(@"self class:%@",[self class]);  

```

在建立``KVO``监听前，打印结果为：

```
self->isa:A
self class:A
```


在建立``KVO``监听之后，打印结果为：

```
self->isa:NSKVONotifying_A
self class:A

```

在这个过程，被观察对象的 ``isa`` 指针从指向原来的 ``A`` 类，被``KVO``机制修改为指向系统新创建的子类NSKVONotifying_A 类，来实现当前类属性值改变的监听；
所以当我们从应用层面上看来，完全没有意识到有新的类出现，这是系统“隐瞒”了对 KVO 的底层实现过程，让我们误以为还是原来的类。但是此时如果我们创建一个新的名为“NSKVONotifying_A”的类，就会发现系统运行到注册 KVO 的那段代码时程序就崩溃，因为系统在注册监听的时候动态创建了名为 NSKVONotifying_A 的中间类，并指向这个中间类了。



* 子类setter方法剖析
``KVO`` 的键值观察通知依赖于 NSObject 的两个方法:``willChangeValueForKey:``和 ``didChangeValueForKey:`` ，在存取数值的前后分别调用 2 个方法：
被观察属性发生改变之前，``willChangeValueForKey:``被调用，通知系统该 ``keyPath`` 的属性值即将变更；
当改变发生后， ``didChangeValueForKey:`` 被调用，通知系统该``keyPath ``的属性值已经变更；之后， ``observeValueForKey:ofObject:change:context:``也会被调用。且重写观察属性的setter 方法这种继承方式的注入是在运行时而不是编译时实现的。

KVO 为子类的观察者属性重写调用存取方法的工作原理在代码中相当于：

```
- (void)setName:(NSString *)newName { 
      [self willChangeValueForKey:@"name"];    //KVO 在调用存取方法之前总调用 
      [super setValue:newName forKey:@"name"]; //调用父类的存取方法 
      [self didChangeValueForKey:@"name"];     //KVO 在调用存取方法之后总调用
}
```

### 消息转发(热更新)解决Bug(JSPatch)

>JSPatch 是一个 iOS 动态更新框架，只需在项目中引入极小的引擎，就可以使用 JavaScript 调用任何 Objective-C 原生接口，获得脚本语言的优势：为项目动态添加模块，或替换项目原生代码动态修复 bug。
>

关于消息转发，前面已经讲到过了，消息转发分为三级，我们可以在每级实现替换功能，实现消息转发，从而不会造成崩溃。JSPatch不仅能够实现消息转发，还可以实现方法添加、替换能一系列功能。


### 实现NSCoding的自动归档和自动解档

原理描述：用``runtime``提供的函数遍历``Model``自身所有属性，并对属性进行``encode``和``decode``操作。
核心方法：在``Model``的基类中重写方法：

```
- (id)initWithCoder:(NSCoder *)aDecoder {
    if (self = [super init]) {
        unsigned int outCount;
        Ivar * ivars = class_copyIvarList([self class], &outCount);
        for (int i = 0; i < outCount; i ++) {
            Ivar ivar = ivars[i];
            NSString * key = [NSString stringWithUTF8String:ivar_getName(ivar)];
            [self setValue:[aDecoder decodeObjectForKey:key] forKey:key];
        }
    }
    return self;
}

- (void)encodeWithCoder:(NSCoder *)aCoder {
    unsigned int outCount;
    Ivar * ivars = class_copyIvarList([self class], &outCount);
    for (int i = 0; i < outCount; i ++) {
        Ivar ivar = ivars[i];
        NSString * key = [NSString stringWithUTF8String:ivar_getName(ivar)];
        [aCoder encodeObject:[self valueForKey:key] forKey:key];
    }
}


```


### 实现字典和模型的自动转换(MJExtension)

原理描述：用``runtime``提供的函数遍历Model自身所有属性，如果属性在json中有对应的值，则将其赋值。
核心方法：在NSObject的分类中添加方法

```
- (instancetype)initWithDict:(NSDictionary *)dict {

    if (self = [self init]) {
        //(1)获取类的属性及属性对应的类型
        NSMutableArray * keys = [NSMutableArray array];
        NSMutableArray * attributes = [NSMutableArray array];
        /*
         * 例子
         * name = value3 attribute = T@"NSString",C,N,V_value3
         * name = value4 attribute = T^i,N,V_value4
         */
        unsigned int outCount;
        objc_property_t * properties = class_copyPropertyList([self class], &outCount);
        for (int i = 0; i < outCount; i ++) {
            objc_property_t property = properties[i];
            //通过property_getName函数获得属性的名字
            NSString * propertyName = [NSString stringWithCString:property_getName(property) encoding:NSUTF8StringEncoding];
            [keys addObject:propertyName];
            //通过property_getAttributes函数可以获得属性的名字和@encode编码
            NSString * propertyAttribute = [NSString stringWithCString:property_getAttributes(property) encoding:NSUTF8StringEncoding];
            [attributes addObject:propertyAttribute];
        }
        //立即释放properties指向的内存
        free(properties);

        //(2)根据类型给属性赋值
        for (NSString * key in keys) {
            if ([dict valueForKey:key] == nil) continue;
            [self setValue:[dict valueForKey:key] forKey:key];
        }
    }
    return self;

}


```








