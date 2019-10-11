# 静态库-动态库简介


## 什么是库？

* 库是共享程序代码的方式，一般分为静态库和动态库。


## 静态库与动态库的区别？

* 静态库：静态库即静态链接库（Windows 下的 .lib，Linux 和 Mac 下的 .a）。之所以叫做静态，是因为静态库在编译的时候会被直接拷贝一份，复制到目标程序里，这段代码在目标程序里就不会再改变了。

	* 链接时完整地拷贝至可执行文件中，被多次使用就有多份冗余拷贝。
	* 静态库的好处很明显，编译完成之后，库文件实际上就没有作用了。目标程序没有外部依赖，直接就可以运行。当然其缺点也很明显，就是会使用目标程序的体积增大。

* 动态库：动态库即动态链接库（Windows 下的 .dll，Linux 下的 .so，Mac 下的 .dylib/.tbd）。与静态库相反，动态库在编译时并不会被拷贝到目标程序中，目标程序中只会存储指向动态库的引用。等到程序运行时，动态库才会被真正加载进来。
	* 链接时不复制，程序运行时由系统动态加载到内存，供程序调用，系统只加载一次，多个程序共用，节省内存。
	* 动态库的优点是，不需要拷贝到目标程序中，不会影响目标程序的体积，而且同一份库可以被多个程序使用（因为这个原因，动态库也被称作共享库）。同时，编译时才载入的特性，也可以让我们随时对库进行替换，而不需要重新编译代码。动态库带来的问题主要是，动态载入会带来一部分性能损失，使用动态库也会使得程序依赖于外部环境。如果环境缺少动态库或者库的版本不正确，就会导致程序无法运行（Linux 下喜闻乐见的 lib not found 错误）。


###  iOS里静态库形式？

* .a和.framework

### iOS里动态库形式？

* .dylib和.framework


### framework为什么既是静态库又是动态库？

* 系统的.framework是动态库，我们自己建立的.framework是静态库。
* 在 iOS 8 之前，iOS 平台不支持使用动态 Framework，开发者可以使用的 Framework 只有苹果自家的 UIKit.Framework，Foundation.Framework 等。这种限制可能是出于安全的考虑。换一个角度讲，因为 iOS 应用都是运行在沙盒当中，不同的程序之间不能共享代码，同时动态下载代码又是被苹果明令禁止的，没办法发挥出动态库的优势，实际上动态库

* iOS 8/Xcode 6 推出之后，iOS 平台添加了动态库的支持，同时 Xcode 6 也原生自带了 Framework 支持（动态和静态都可以），上面提到的的奇技淫巧也就没有必要了（新的做法参考这里）。为什么 iOS 8 要添加动态库的支持？唯一的理由大概就是 Extension 的出现。Extension 和 App 是两个分开的可执行文件，同时需要共享代码，这种情况下动态库的支持就是必不可少的了。但是这种动态 Framework 和系统的 UIKit.Framework 还是有很大区别。系统的 Framework 不需要拷贝到目标程序中，我们自己做出来的 Framework 哪怕是动态的，最后也还是要拷贝到 App 中（App 和 Extension 的 Bundle 是共享的），因此苹果又把这种 Framework 称为 [Embedded Framework](https://developer.apple.com/library/archive/documentation/General/Conceptual/ExtensibilityPG/ExtensionScenarios.html)。

## .a与.framework有什么区别


* .a是一个纯二进制文件，.framework中除了有二进制文件之外还有资源文件。


* .a文件不能直接使用，至少要有.h文件配合，.framework文件可以直接使用。

* .a + .h + sourceFile = .framework。

* 建议用.framework.



## 为什么要使用静态库？

* 方便共享代码，便于合理使用。
* 实现iOS程序的模块化。可以把固定的业务模块化成静态库。
* 和别人分享你的代码库，但不想让别人看到你代码的实现。
* 开发第三方sdk的需要。


## 制作静态库时的几点注意：

*  注意理解：无论是.a静态库还.framework静态库，我们需要的都是二进制文件+.h+其它资源文件的形式，不同的是，.a本身就是二进制文件，需要我们自己配上.h和其它文件才能使用，而.framework本身已经包含了.h和其它文件，可以直接使用。


* 图片资源的处理：两种静态库，一般都是把图片文件单独的放在一个.bundle文件中，一般.bundle的名字和.a或.framework的名字相同。.bundle文件很好弄，新建一个文件夹，把它改名为.bundle就可以了，右键，显示包内容可以向其中添加图片资源。

* category是我们实际开发项目中经常用到的，把category打成静态库是没有问题的，但是在用这个静态库的工程中，调用category中的方法时会有找不到该方法的运行时错误（selector not recognized），解决办法是：在使用静态库的工程中配置other linker flags的值为-ObjC

* 如果一个静态库很复杂，需要暴露的.h比较多的话，就可以在静态库的内部创建一个.h文件（一般这个.h文件的名字和静态库的名字相同），然后把所有需要暴露出来的.h文件都集中放在这个.h文件中，而那些原本需要暴露的.h都不需要再暴露了，只需要把.h暴露出来就可以了。


## Swift 支持

跟着 iOS8 / Xcode 6 同时发布的还有 Swift。如果要在项目中使用外部的代码，可选的方式只有两种，一种是把代码拷贝到工程中，另一种是用动态 Framework。使用静态库是不支持的。

造成这个问题的原因主要是 Swift 的运行库没有被包含在 iOS 系统中，而是会打包进 App 中（这也是造成 Swift App 体积大的原因），静态库会导致最终的目标程序中包含重复的运行库（这是[苹果自家的解释](https://github.com/ksm/SwiftInFlux#static-libraries)）。同时拷贝 Runtime 这种做法也会导致在纯 ObjC 的项目中使用 Swift 库出现问题。苹果声称等到 Swift 的 Runtime 稳定之后会被加入到系统当中，到时候这个限制就会被去除了（参考[这个问题](https://stackoverflow.com/questions/25020783/how-to-distribute-swift-library-without-exposing-the-source-code) 的问题描述，也是来自苹果自家文档）。


## CocoaPods 的做法

纯 ObjC 的项目中，CocoaPods 使用编译静态库 .a 方法将代码集成到项目中。在 Pods 项目中的每个 target 都对应这一个 Pod 的静态库。不过在编译过程中并不会真的产出 .a 文件。如果需要 .a 文件的话,使用 CocoasPods-Packager 这个插件。

当不想发布代码的时候，也可以使用 Framework 发布 Pod，CocoaPods 提供了 vendored_framework 选项来使用第三方 Framework

对于 Swift 项目，CocoaPods 提供了动态 Framework 的支持。通过 use_frameworks! 选项控制。对于 Swift 写的库来说，想通过 CocoaPods 引入工程，必须加入 use_frameworks! 选项

