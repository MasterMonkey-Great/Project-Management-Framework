# 第八章 项目工程二进制化方案



随着业务的扩展，私有CocoaPod库和第三方 CocoaPod 库越来越多，App项目中的文件也越来越多。每次 pod install/update 或提交到 Jenkins 上构建的时候，重新编译的过程需要等待很长时间，这就间接地向我们提出了加快编译速度的需求。


### 什么是组件二进制化？


开发过程中我们使用CocoaPods生成、管理和使用模块库、组件库或业务库，在工程中组件库或业务库均可以看成一个模块。二进制化指的是通过编译把模块的源码转换成静态库或动态库，以提高该组件在App项目中的编译速度。


