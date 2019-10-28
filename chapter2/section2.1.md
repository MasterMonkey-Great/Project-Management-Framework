# 2.1 Cocoapods安装


## 环境搭建

####查看当前 ruby 与 gem 版本

```
MacBook-Pro:coo$ ruby -v
ruby 2.2.6p396 (2016-11-15 revision 56800) [x86_64-darwin17]
MacBook-Pro:coo$ gem -v
2.7.7
```

#### ruby 与 gem需要更新到最新 切换源路径或者使用VPN
 * sudo gem update --system:更新gem
 * brew install ruby  ：更新ruby
 * 更换源（因为Ruby的软件源rubygems.org被屏蔽了，需要来修改更换源，把源切换至ruby-china；网上大多数是使用的https://ruby.taobao.org的，这里不再建议使用的了，这是因为taobao Gems 源已停止维护，现由 ruby-china 提供镜像服务）

更换源命令 ：
```
gem sources --add https://gems.ruby-china.com/ --remove https://rubygems.org/
```

接下来查看源路径是否替换成功，执行命令：

```
gem sources -l
```


## 安装Cocoapods

#### 执行安装命令

```
sudo gem install cocoapods
```

* 显示如下错误 

```
ERROR: While executing gem … (Gem::FilePermissionError) 
You don’t have write permissions for the /usr/bin directory.
```

* 升级macOS10.13.4之后，cocoapods不能正常使用了，苹果一贯的问题，遇到大版本系统升级，之前的一些软件就不能正常使用了。


#### 重新安装 输入一下命令

```
sudo gem install -n /usr/local/bin sass 
sudo gem install -n /usr/local/bin cocoapods
```


####查看Cocoapods 是否安装成功

```
pod --version
```

####  Cocoapods本地索引库克隆

```
pod setup
```

* 执行结果


```
MacBook-Pro:~ coo$ pod setup
Setting up CocoaPods master repo
$ /usr/bin/git clone https://github.com/CocoaPods/Specs.git master --progress
Cloning into 'master'...
remote: Counting objects: 2278769, done.        
remote: Compressing objects: 100% (374/374), done.        
error: RPC failed; curl 56 LibreSSL SSL_read: SSL_ERROR_SYSCALL, errno 54
fatal: The remote end hung up unexpectedly
fatal: early EOF
fatal: index-pack failed
[!] /usr/bin/git clone https://github.com/CocoaPods/Specs.git master --progress
	
Cloning into 'master'...
remote: Counting objects: 2278769, done.        
remote: Compressing objects: 100% (374/374), done.        
error: RPC failed; curl 56 LibreSSL SSL_read: SSL_ERROR_SYSCALL, errno 54
fatal: The remote end hung up unexpectedly
fatal: early EOF
fatal: index-pack failed

```



#### Cocoapods初始化失败解决方案

* 打开终端命令行，输入一下命令：

```
sudo xcode-select -switch /Applications/Xcode.app/Contents/Developer
```

* 当然一个xcode执行上文命令行肯定没问题，如果多个

```
sudo xcode-select -switch /Applications/Xcode 7.3.1.app/Contents/Developer
```

* 最简便的方式 : 先在终端输入```sudo xcode-select -switch```，然后，打开Xcode—>右键显示包内容，找到Developer文件夹拖到终端里面

第三种方案执行结果如下

```
MacBook-Pro:~ coo$ sudo xcode-select -switch /Applications/Xcode.app/Contents/Developer 
Password:
MacBook-Pro:~ coo$ pod setup
Setting up CocoaPods master repo
$ /usr/bin/git clone https://github.com/CocoaPods/Specs.git master --progress
Cloning into 'master'...
remote: Counting objects: 2278814, done.        
remote: Compressing objects: 100% (418/418), done.        
remote: Total 2278814 (delta 168), reused 35 (delta 35), pack-reused 2278345        
Receiving objects: 100% (2278814/2278814), 553.14 MiB | 531.00 KiB/s, done.
Resolving deltas:  98% (1283327/1309475)   
Resolving deltas: 100% (1309475/1309475), done.
Setup completed

```


## Cocoapods应用原理剖析

####图解Cocoapods工作流程

后续加上


####Cocoapods中重要的路径

* pod远程索引库：``` https://github.com/CocoaPods/Specs.git ```
* pod本地索引库：``` /Users/coo/.cocoapods/repos/master/Specs ```
* 本地检索索引文件：``` /Users/coo/Library/Caches/CocoaPods/search_index.json  ```


###Cocoapods 终端命令行使用

* pod setup  
* pod --version: 查看当前pod 版本
* pod init:本地项目pod初始化，自动生成格式化Podfile文件
* pod search MJExtension ：查询pod中该远程公有组件情况
* pod install :更新本地工程pod下第三方组件，依据Podfile.lock文件(工作中尽量使用这个)
* pod update:更新本地工程pod下第三方组件，依据Podfile文件,更新Podfile.lock文件

* pod --help


######pod --help 执行结果

```
Commands:

+ cache         Manipulate the CocoaPods cache
+ deintegrate   Deintegrate CocoaPods from your project
+ env           Display pod environment
+ init          Generate a Podfile for the current directory
+ install       Install project dependencies according to versions from a
Podfile.lock
+ ipc           Inter-process communication
+ lib           Develop pods
+ list          List pods
+ outdated      Show outdated project dependencies
+ plugins       Show available CocoaPods plugins
+ repo          Manage spec-repositories
+ search        Search for pods
+ setup         Setup the CocoaPods environment
+ spec          Manage pod specs
+ trunk         Interact with the CocoaPods API (e.g. publishing new specs)
+ try           Try a Pod!
+ update        Update outdated project dependencies and create new
Podfile.lock

```