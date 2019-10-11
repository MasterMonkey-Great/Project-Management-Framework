# 第十六章 Method Swizzling

###Objective-C 中的 Hook 又被称作 Method Swizzling，这是动态语言大都具有的特性。在 Objective-C 中经常会把 Hook 的逻辑写在 + load 方法中，这是利用它调用时机较提前等性质。


#####有时候需要 Hook 子类和父类的同一个方法，但是它们的 + load 方法调用顺序不同。一个常见的顺序可能是：父类->子类->子类类别->父类类别。所以 Hook 的顺序并不能保证，就不能保证 Hook 后方法调用的顺序是对的。而且使用不同方法 Method Swizzling 也会带来不同的结果。本文将会对这些情况下的 Hook 结果进行分析和总结。



目前有两类常用的 Method Swizzling 实现方案，诸如 [RSSwizzle](https://github.com/rabovik/RSSwizzle) 和 [jrswizzle](https://github.com/rentzsch/jrswizzle) 这种较为复杂且周全的一些实现方案。
