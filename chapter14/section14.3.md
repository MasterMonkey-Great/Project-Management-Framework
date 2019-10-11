# 14.3  API 


## Class相关API

```
// Person类继承自NSObject，包含run方法
@interface Person : NSObject
@property (nonatomic, strong) NSString *name;
- (void)run;
@end

#import "Person.h"
@implementation Person
- (void)run
{
    NSLog(@"%s",__func__);
}
@end

// Car类继承自NSObejct，包含run方法
#import "Car.h"
@implementation Car
- (void)run
{
    NSLog(@"%s",__func__);
}
@end



```




1. 动态创建一个类（参数：父类，类名，额外的内存空间）

	```
	Class objc_allocateClassPair(Class superclass, const char *name, size_t extraBytes)
	```

2. 注册一个类（要在类注册之前添加成员变量）

	```
	void objc_registerClassPair(Class cls) 
	
	```

3. 销毁一个类

	```
	void objc_disposeClassPair(Class cls)
	
	示例：
	void run(id self , SEL _cmd) {
	    NSLog(@"%@ - %@", self,NSStringFromSelector(_cmd));
	}
	
	int main(int argc, const char * argv[]) {
	    @autoreleasepool {
	        // 创建类 superclass:继承自哪个类 name:类名 size_t:格外的大小，创建类是否需要扩充空间
	        // 返回一个类对象
	        Class newClass = objc_allocateClassPair([NSObject class], "Student", 0);
	        
	        // 添加成员变量 
	        // cls:添加成员变量的类 name:成员变量的名字 size:占据多少字节 alignment:内存对齐，最好写1 types:类型，int类型就是@encode(int) 也就是i
	        class_addIvar(newClass, "_age", 4, 1, @encode(int));
	        class_addIvar(newClass, "_height", 4, 1, @encode(float));
	        
	        // 添加方法
	        class_addMethod(newClass, @selector(run), (IMP)run, "v@:");
	        
	        // 注册类
	        objc_registerClassPair(newClass);
	        
	        // 创建实例对象
	        id student = [[newClass alloc] init];
	    
	        // 通过KVC访问
	        [student setValue:@10 forKey:@"_age"];
	        [student setValue:@180.5 forKey:@"_height"];
	        
	        // 获取成员变量
	        NSLog(@"_age = %@ , _height = %@",[student valueForKey:@"_age"], [student valueForKey:@"_height"]);
	        
	        // 获取类的占用空间
	        NSLog(@"类对象占用空间%zd", class_getInstanceSize(newClass));
	        
	        // 调用动态添加的方法
	        [student run];
	        
	    }
	    return 0;
	}
	
	// 打印内容
	// Runtime应用[25605:4723961] _age = 10 , _height = 180.5
	// Runtime应用[25605:4723961] 类对象占用空间16
	// Runtime应用[25605:4723961] <Student: 0x10072e420> - run
	
	注意
	类一旦注册完毕，就相当于类对象和元类对象里面的结构就已经创建好了。
	因此必须在注册类之前，添加成员变量。方法可以在注册之后再添加，因为方法是可以动态添加的。
	创建的类如果不需要使用了 ，需要释放类。
	
	
	```
	


4. 获取isa指向的Class，如果将类对象传入获取的就是元类对象，如果是实例对象则为类对象

	```
	Class object_getClass(id obj)
	
	int main(int argc, const char * argv[]) {
	    @autoreleasepool {
	        Person *person = [[Person alloc] init];
	        NSLog(@"%p,%p,%p",object_getClass(person), [Person class],
	              object_getClass([Person class]));
	    }
	    return 0;
	}
	// 打印内容
	Runtime应用[21115:3807804] 0x100001298,0x100001298,0x100001270
	
	
	```



5. 设置isa指向的Class，可以动态的修改类型。例如修改了person对象的类型，也就是说修改了person对象的isa指针的指向，中途让对象去调用其他类的同名方法。

	```
	Class object_setClass(id obj, Class cls)
	
	int main(int argc, const char * argv[]) {
	    @autoreleasepool {
	        Person *person = [[Person alloc] init];
	        [person run];
	        
	        object_setClass(person, [Car class]);
	        [person run];
	    }
	    return 0;
	}
	// 打印内容
	Runtime应用[21147:3815155] -[Person run]
	Runtime应用[21147:3815155] -[Car run]
	最终其实调用了car的run方法
	
	```



6. 用于判断一个OC对象是否为Class

	```
	BOOL object_isClass(id obj)
	
	// 判断OC对象是实例对象还是类对象
	NSLog(@"%d",object_isClass(person)); // 0
	NSLog(@"%d",object_isClass([person class])); // 1
	NSLog(@"%d",object_isClass(object_getClass([person class]))); // 1 
	// 元类对象也是特殊的类对象
	
	```

7. 判断一个Class是否为元类
	
	```
	
	BOOL class_isMetaClass(Class cls)
	
	
	```
8. 获取类对象父类

	```
	Class class_getSuperclass(Class cls)
	```

## 成员变量相关API


1. 获取一个实例变量信息，描述信息变量的名字，占用多少字节等

	```
	Ivar class_getInstanceVariable(Class cls, const char *name)
	```

2. 拷贝实例变量列表（最后需要调用free释放）

	```
	Ivar *class_copyIvarList(Class cls, unsigned int *outCount)
	```

3. 设置和获取成员变量的值

	```
	void object_setIvar(id obj, Ivar ivar, id value)
	id object_getIvar(id obj, Ivar ivar)
	```

4. 动态添加成员变量（已经注册的类是不能动态添加成员变量的）

	```
	BOOL class_addIvar(Class cls, const char * name, size_t size, uint8_t alignment, const char * types)
	```

5. 获取成员变量的相关信息，传入成员变量信息，返回C语言字符串

	```
	const char *ivar_getName(Ivar v)
	```

6. 获取成员变量的编码，types

	```
	const char *ivar_getTypeEncoding(Ivar v)
	
	示例：
	int main(int argc, const char * argv[]) {
	    @autoreleasepool {
	        // 获取成员变量的信息
	        Ivar nameIvar = class_getInstanceVariable([Person class], "_name");
	        // 获取成员变量的名字和编码
	        NSLog(@"%s, %s", ivar_getName(nameIvar), ivar_getTypeEncoding(nameIvar));
	        
	        Person *person = [[Person alloc] init];
	        // 设置和获取成员变量的值
	        object_setIvar(person, nameIvar, @"xx_cc");
	        // 获取成员变量的值
	        object_getIvar(person, nameIvar);
	        NSLog(@"%@", object_getIvar(person, nameIvar));
	        NSLog(@"%@", person.name);
	        
	        // 拷贝实例变量列表
	        unsigned int count ;
	        Ivar *ivars = class_copyIvarList([Person class], &count);
	
	        for (int i = 0; i < count; i ++) {
	            // 取出成员变量
	            Ivar ivar = ivars[i];
	            NSLog(@"%s, %s", ivar_getName(ivar), ivar_getTypeEncoding(ivar));
	        }
	        
	        free(ivars);
	
	    }
	    return 0;
	}

	// 打印内容
	// Runtime应用[25783:4778679] _name, @"NSString"
	// Runtime应用[25783:4778679] xx_cc
	// Runtime应用[25783:4778679] xx_cc
	// Runtime应用[25783:4778679] _name, @"NSString"
	
	
	
	```
## 属性相关API

1. 获取一个属性

	```
	objc_property_t class_getProperty(Class cls, const char *name)
	```

2. 拷贝属性列表（最后需要调用free释放）

	```
	objc_property_t *class_copyPropertyList(Class cls, unsigned int *outCount)
	```

3. 动态添加属性

	```
	BOOL class_addProperty(Class cls, const char *name, const objc_property_attribute_t *attributes,
	                  unsigned int attributeCount)
	
	```

4. 动态替换属性

	```
	void class_replaceProperty(Class cls, const char *name, const objc_property_attribute_t *attributes,
	                      unsigned int attributeCount)
	```



5. 获取属性的一些信息

	```
	const char *property_getName(objc_property_t property)
	const char *property_getAttributes(objc_property_t property)
	```

## 方法相关API

1. 获得一个实例方法、类方法

	```
	Method class_getInstanceMethod(Class cls, SEL name)
	Method class_getClassMethod(Class cls, SEL name)
	```

2. 方法实现相关操作

	```
	IMP class_getMethodImplementation(Class cls, SEL name) 
	IMP method_setImplementation(Method m, IMP imp)
	void method_exchangeImplementations(Method m1, Method m2) 
	```


3. 拷贝方法列表（最后需要调用free释放）

	```
	Method *class_copyMethodList(Class cls, unsigned int *outCount)
	```

4. 动态添加方法

	```
	BOOL class_addMethod(Class cls, SEL name, IMP imp, const char *types)
	```


5. 动态替换方法

	```
	IMP class_replaceMethod(Class cls, SEL name, IMP imp, const char *types)
	```

6. 获取方法的相关信息（带有copy的需要调用free去释放）

	```
	SEL method_getName(Method m)
	IMP method_getImplementation(Method m)
	const char *method_getTypeEncoding(Method m)
	unsigned int method_getNumberOfArguments(Method m)
	char *method_copyReturnType(Method m)
	char *method_copyArgumentType(Method m, unsigned int index)
	```
	

7. 选择器相关

	```
	const char *sel_getName(SEL sel)
	SEL sel_registerName(const char *str)
	```

8. 用block作为方法实现

	```
	IMP imp_implementationWithBlock(id block)
	id imp_getBlock(IMP anImp)
	BOOL imp_removeBlock(IMP anImp)
	```



