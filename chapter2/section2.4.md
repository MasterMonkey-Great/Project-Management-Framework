# 2.4 Cocoapods .podspec 说明文件编写


[官方.podspec 使用方法](https://guides.cocoapods.org/syntax/podspec.html)



## pod lib create XXX 样板说明文件

```

#
# Be sure to run `pod lib lint MeCenterKit.podspec' to ensure this is a
# valid spec before submitting.
#
# Any lines starting with a # are optional, but their use is encouraged
# To learn more about a Podspec see https://guides.cocoapods.org/syntax/podspec.html
#

Pod::Spec.new do |s|
  s.name             = 'MeCenterKit'
  s.version          = '0.1.0'
  s.summary          = 'A short description of MeCenterKit.'

# This description is used to generate tags and improve search results.
#   * Think: What does it do? Why did you write it? What is the focus?
#   * Try to keep it short, snappy and to the point.
#   * Write the description between the DESC delimiters below.
#   * Finally, don't worry about the indent, CocoaPods strips it!

  s.description      = <<-DESC
TODO: Add long description of the pod here.
                       DESC

  s.homepage         = 'https://github.com/beichenqing528369/MeCenterKit'
  # s.screenshots     = 'www.example.com/screenshots_1', 'www.example.com/screenshots_2'
  s.license          = { :type => 'MIT', :file => 'LICENSE' }
  s.author           = { 'beichenqing528369' => 'chenbo@88gongxiang.com' }
  s.source           = { :git => 'https://github.com/beichenqing528369/MeCenterKit.git', :tag => s.version.to_s }
  # s.social_media_url = 'https://twitter.com/<TWITTER_USERNAME>'

  s.ios.deployment_target = '8.0'

  s.source_files = 'MeCenterKit/Classes/**/*'
  
  # s.resource_bundles = {
  #   'MeCenterKit' => ['MeCenterKit/Assets/*.png']
  # }

  # s.public_header_files = 'Pod/Classes/**/*.h'
  # s.frameworks = 'UIKit', 'MapKit'
  # s.dependency 'AFNetworking', '~> 2.3'
end


```


* 必要属性

```
Pod::Spec.new do|s|
  //项目名
  s.name ='HycProject'
  //版本号
  s.version ='1.0.1'
  //license文件的类型
  s.license = 'MIT'
  //简单描述
  s.summary = 'ATest in iOS.'
  //项目的getub地址，只支持HTTP和HTTPS地址，不支持ssh的地址
  s.homepage ='https://github.com/hyc603671932/HycProject'
  //作者和邮箱
  s.authors = {'HouKavin' => '603671932@qq.com' }
  //git仓库的https地址
  s.source = { :git=> 'https://github.com/hyc603671932/HycProject.git', :tag =>s.version}
  //是否要求arc（有部分非arc文件情况未考证）
  s.requires_arc = true
 //在这个属性中声明过的.h文件能够使用<>方法联想调用（这个是可选属性）
  s.public_header_files = 'UIKit/*.h'
  //表示源文件的路径，这个路径是相对podspec文件而言的。（这属性下面单独讨论）
  s.source_files ='AppInfo/*.*'
  //需要用到的frameworks，不需要加.frameworks后缀。（这个没有用到也可以不填）
  s.frameworks ='Foundation', 'CoreGraphics', 'UIKit'
end

```


* 可选属性

```
Pod::Spec.new do|s|
  ...
  ...
  ...
  //详细介绍
  s.description = "详细介绍"
  //支持的平台及版本
  s.platform     = :ios, '7.0' 
  //最低要求的系统版本
  s.ios.deployment_target= '7.0'
  //主页，需填写可访问地址
  s.homepage = "https://coding.net/u/wtlucky/p/podTestLibrary"
  //截图                      
  s.screenshots     = "www.example.com/screenshots_1"
  //多媒体介绍地址
  s.social_media_url = 'https://twitter.com/<twitter_username>'  
  //效果和s.public_header_files的相同，只需要配置一种
  s.ios.public_header_files = 'URS/URSAuth.h'
  //不常用，所有文件默认即为private只能用import"XXX"调用
  s.ios.private_header_files 
  #依赖关系，该项目所依赖的其他库
  s.dependency 'AFNetworking', '~> 2.3'
  //可拥有多个dependency依赖属性   
  s.dependency 'JSONKit', '~> 1.4' 

  //动态库所使用的资源文件存放位置，放在Resources文件夹中
  s.resource_bundles = {
    'Test5' => ['Test5/Assets/*.png']
  } 
  //资源文件（具体使用带考证）
  s.resources = 'src/SinaWeibo/SinaWeibo.bundle/**/*.png' 
  //建立名称为Info的子文件夹（虚拟路径）
  s.subspec 'Info' do |ss| 
  //应该和s.subspec作用相同（未考证）
  s.default_subspec='Core'
  //多个子subspec
  s.default_subspec = ['EmptyView', 'PlaceholderTextView', 'UserManager', 'CellAnimator', 'KFImageProcessor']

end

```

* 库资源文件封装引用

```
//下载AppInfo文件夹下的所有文件，子文件夹不识别
s.source_files ='AppInfo'
//下载AppInfo目录下所有格式文件
s.source_files ='AppInfo/*.*'
**/*表示Classes目录及其子目录下所有文件
s.source_files = 'AppInfo/**/*'


//下载HycProject文件夹下名称为AppManInfo和AppWomanInfo的共4项文件
s.source_files ='HycProject/App{Man,Woman}Info.{h,m}'
//目标路径下的文件不进行下载
s.ios.exclude_files = 'AppInfo/Info/json'

```

* 库资源文件层次

```
Pod::Spec.new do|s|
  //第一层文件夹名称HycProject
  s.name ='HycProject'
  ...
  ...
  ...
  //第二层文件夹名称AppInfo（虚拟路径）
  s.subspec 'AppInfo' do |ss|
    //下载HycProject文件夹下AppInfo的.h和.m文件
    ss.source_files = 'HycProject/AppInfo.{h,m}'
    //允许使用import<AppInfo.h>
    ss.public_header_files = 'HycProject/AppInfo.h'
    //依赖的frameworks
    ss.ios.frameworks = 'MobileCoreServices', 'CoreGraphics'

    //第三层文件夹名称Info（虚拟路径）
    ss.subspec 'Info' do |sss| 
      //最低要求的系统版本7.0
      sss.ios.deployment_target = '7.0' 
      //只允许使用import"AppInfo.h"访问
      sss.ios.private_header_files = 'AppInfo/Info/**/*.h' 
      // 下载路径下的.h/.m/.c文件       
      sss.ios.source_files = 'AppInfo/Info/**/*.{h,m,c}'  
      //引用xml2库,但系统会找不到这个库的头文件，需与下方sss.xcconfig配合使用(这里省略lib)
      sss.libraries = "xml2" 
      //在pod target项的Header Search Path中配置:${SDK_DIR}/usr/include/libxml2
      sss.xcconfig = { 'HEADER_SEARCH_PATHS' => '${SDK_DIR}/usr/include/libxml2' }       
      //json目录下的文件不做下载
      sss.ios.exclude_files = 'AppInfo/Info/json' 
    end   
  end
end

```



* framworks  库需要添加的系统库

```
spec.ios.framework = 'CFNetwork'
spec.frameworks = 'QuartzCore', 'CoreData'

```


*  weak_frameworks


```
spec.weak_framework = 'Twitter'
spec.weak_frameworks = 'Twitter', 'SafariServices'

```

* 引用库文件

```
静态库.a spec.vendored_libraries  = 'NCKFoundation/Classes/ThirdParty/*.{a}'

framework库 s.vendored_frameworks =
  'Pod/Classes/ZMCreditSDK.framework’ ,
  'Pod/Classes/ZMDependUponSDK.framework'

```



