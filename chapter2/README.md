# 第二章 Cocoapods基础功能

###CocoaPods简介

CocoaPods是OS X和iOS下的一个第三类库管理工具，通过CocoaPods工具我们可以为项目添加被称为“Pods”的依赖库（这些类库必须是CocoaPods本身所支持的），并且可以轻松管理其版本。

1. 在引入第三方库时它可以自动为我们完成各种各样的配置，包括配置编译阶段、连接器选项、甚至是ARC环境下的-fno-objc-arc配置等。
2. 使用CocoaPods可以很方便地查找新的第三方库，这些类库是比较“标准的”，而不是网上随便找到的，这样可以让我们找到真正好用的类库。



### iOS库分类介绍

IOS工程有3种库项目，framework，static library，meta library，通常只使用前两种。我们在使用static library库工程时，一般使用它编译出来的静态库libxxx.a，以及对应的头文件，在写应用时，将这些文件拷贝到项目里，然后将静态库添加到链接的的依赖库路径里，并将头文件目录添加到头文件搜索目录中。而framework库的依赖会简单很多，framework是资源的集合，将静态库和其头文件包含在framework目录里。


### 核心组件

CocoaPods是用 Ruby 写的，并由若干个 Ruby 包 (gems) 构成的。在解析整合过程中，最重要的几个 gems 分别是： CocoaPods/CocoaPods, CocoaPods/Core, 和 CocoaPods/Xcodeproj (是的，CocoaPods 是一个依赖管理工具 -- 利用依赖管理进行构建的！)。

```
CocoaPods 是一个 objc 的依赖管理工具，而其本身是利用 ruby 的依赖管理 gem 进行构建的

```

* CocoaPods/CocoaPod

这是是一个面向用户的组件，每当执行一个 pod 命令时，这个组件都将被激活。该组件包括了所有使用 CocoaPods 涉及到的功能，并且还能通过调用所有其它的 gems 来执行任务。

* CocoaPods/Core

Core 组件提供支持与 CocoaPods 相关文件的处理，文件主要是 Podfile 和 podspecs。

* Podfile

Podfile 是一个文件，用于定义项目所需要使用的第三方库。该文件支持高度定制，你可以根据个人喜好对其做出定制

* Podspec

.podspec 也是一个文件，该文件描述了一个库是怎样被添加到工程中的。它支持的功能有：列出源文件、framework、编译选项和某个库所需要的依赖等。

* CocoaPods/Xcodeproj

这个 gem 组件负责所有工程文件的整合。它能够对创建并修改 .xcodeproj 和 .xcworkspace 文件。它也可以作为单独的一个 gem 包使用。如果你想要写一个脚本来方便的修改工程文件，那么可以使用这个 gem。




### CocoaPods配置文件介绍

使用CocoaPods时，肯定会用到下文要介绍的文件

1. PodFile 依赖描述文件

2. Podfile.lock 当前安装的依赖库的版本

3. Demo.xcworkspace
    
    >xcworkspace文件，使用CocoaPod管理依赖的项目，XCode只能使用workspace编译  项目，如果还只打开以前的xcodeproj文件进行开发，编译会失败，xcworkspace文件实际是一个文件夹，实际Workspace信息保存在contents.xcworkspacedata里，该文件的内容非常简单，实际上只指示它所使用的工程的文件目录, 如下
   
   
   ```
   <?xml version="1.0" encoding="UTF-8"?>
<Workspace
   version = "1.0">
   <FileRef
      location = "group:Demo/Demo.xcodeproj">
   </FileRef>
   <FileRef
      location = "group:Pods/Pods.xcodeproj">
   </FileRef>
</Workspace>

   ```
 
### 运行 pod install 命令

当运行 pod install 命令时会引发许多操作。要想深入了解这个命令执行的详细内容，可以在这个命令后面加上 --verbose。现在运行这个命令 ``pod install --verbose``
 
 
### 读取 Podfile 文件

你是否对 Podfile 的语法格式感到奇怪过，那是因为这是用 Ruby 语言写的。相较而言，这要比现有的其他格式更加简单好用一些。

在安装期间，第一步是要弄清楚显示或隐式的声明了哪些第三方库。在加载 podspecs 过程中，CocoaPods 就建立了包括版本信息在内的所有的第三方库的列表。Podspecs 被存储在本地路径 ``~/.cocoapods`` 中。
 


### 版本控制和冲突

CocoaPods 使用语义版本控制 - [Semantic Versioning](https://semver.org) 命名约定来解决对版本的依赖。由于冲突解决系统建立在非重大变更的补丁版本之间，这使得解决依赖关系变得容易很多。例如，两个不同的 pods 依赖于 CocoaLumberjack 的两个版本，假设一个依赖于 2.3.1，另一个依赖于 2.3.3，此时冲突解决系统可以使用最新的版本 2.3.3，因为这个可以向后与 2.3.1 兼容。

但这并不总是有效。有许多第三方库并不使用这样的约定，这让解决方案变得非常复杂。

当然，总会有一些冲突需要手动解决。如果一个库依赖于 CocoaLumberjack 的 1.2.5，另外一个库则依赖于 2.3.1，那么只有最终用户通过明确指定使用某个版本来解决冲突。


### 加载源文件

CocoaPods 执行的下一步是加载源码。每个 .podspec 文件都包含一个源代码的索引，这些索引一般包裹一个 git 地址和 git tag。它们以 commit SHAs 的方式存储在 ``~/Library/Caches/CocoaPods`` 中。这个路径中文件的创建是由 Core gem 负责的。

CocoaPods 将依照 Podfile、.podspec 和缓存文件的信息将源文件下载到 Pods 目录中。


### 生成 Pods.xcodeproj

每次 pod install 执行，如果检测到改动时，CocoaPods 会利用 Xcodeproj gem 组件对 Pods.xcodeproj 进行更新。如果该文件不存在，则用默认配置生成。否则，会将已有的配置项加载至内存中。

### 安装第三方库

当 CocoaPods 往工程中添加一个第三方库时，不仅仅是添加代码这么简单，还会添加很多内容。由于每个第三方库有不同的 target，因此对于每个库，都会有几个文件需要添加，每个 target 都需要：

* 一个包含编译选项的 .xcconfig 文件
* 一个同时包含编译设置和 CocoaPods 默认配置的私有 .xcconfig 文件
* 一个编译所必须的 prefix.pch 文件
* 另一个编译必须的文件 dummy.m
一旦每个 pod 的 target 完成了上面的内容，整个 Pods target 就会被创建。这增加了相同文件的同时，还增加了另外几个文件。如果源码中包含有资源 bundle，将这个 bundle 添加至程序 target 的指令将被添加到 Pods-Resources.sh 文件中。还有一个名为 Pods-environment.h 的文件，文件中包含了一些宏，这些宏可以用来检查某个组件是否来自 pod。最后，将生成两个认可文件，一个是 plist，另一个是 markdown，这两个文件用于给最终用户查阅相关许可信息。



### 写入至磁盘

直到现在，许多工作都是在内存中进行的。为了让这些成果能被重复利用，我们需要将所有的结果保存到一个文件中。所以 ``Pods.xcodeproj`` 文件被写入磁盘，另外两个非常重要的文件：``Podfile.lock`` 和 ``Manifest.lock`` 都将被写入磁盘。


### Podfile.lock

这是 CocoaPods 创建的最重要的文件之一。它记录了需要被安装的 pod 的每个已安装的版本。如果你想知道已安装的 pod 是哪个版本，可以查看这个文件。推荐将 Podfile.lock 文件加入到版本控制中，这有助于整个团队的一致性。


### Manifest.lock

这是每次运行 pod install 命令时创建的 Podfile.lock 文件的副本。如果你遇见过这样的错误 沙盒文件与 Podfile.lock 文件不同步 (The sandbox is not in sync with the Podfile.lock)，这是因为 Manifest.lock 文件和 Podfile.lock 文件不一致所引起。由于 Pods 所在的目录并不总在版本控制之下，这样可以保证开发者运行 app 之前都能更新他们的 pods，否则 app 可能会 crash，或者在一些不太明显的地方编译失败。

### xcproj -- 非常原始的，现在xcode集成类似功能
如果你已经依照我们的建议在系统上安装了 [xcproj](https://github.com/0xced/xcproj)，它会对 Pods.xcodeproj 文件执行一下 touch 以将其转换成为旧的 ASCII plist 格式的文件。为什么要这么做呢？虽然在很久以前就不被其它软件支持了，但是 Xcode 仍然依赖于这种格式。如果没有 xcproj，你的 Pods.xcodeproj 文件将会以 XML 格式的 plist 文件存储，当你用 Xcode 打开它时，它会被改写，并造成大量的文件改动。



###  结果

运行 pod install 命令的最终结果是许多文件被添加到你的工程和系统中。这个过程通常只需要几秒钟。当然没有 Cocoapods 这些事也都可以完成。只不过所花的时间就不仅仅是几秒而已了。







 