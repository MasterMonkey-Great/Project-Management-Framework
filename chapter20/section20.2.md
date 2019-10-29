# 20.2  iOS 音视频开发-MediaPlayer播放本地、远程音频

使用 MediaPlayer 时需要先导入 <MediaPlayer/MediaPlayer.h> 框架头文件。

MediaPlayer 框架是 iOS 平台上一个用于音频和视频播放的高层级接口，它包含了一个你可以在应用中直接使用的默认的用户界面。你可以使用它来播放用户在 iPod 库中的项目，或者播放本地文件以及网络流。

另外，这个框架也包括了查找用户媒体库中内容的 API，同时还可以配置像是在锁屏界面或者控制中心里的音频控件。

[本文中Demo下载](https://github.com/MasterChanMonkey/FF-Media)


### 导入头文件

```
#import <MediaPlayer/MediaPlayer.h>

```

### 声明播放器对象属性

```
@property(strong, nonatomic)MPMoviePlayerController*playerController;

@property(strong, nonatomic)MPMoviePlayerViewController*playerViewController;//播放器对象

```


### MPMoviePlayerController播放本地音频

```
NSURL*url = [[NSBundle mainBundle]URLForResource:@"yxqc.mp3"withExtension:nil];

//  MPMoviePlayerController 需要添加到 View 上

self.playerController= [[MPMoviePlayerController alloc]initWithContentURL:url];

self.playerController.view.frame=self.view.bounds;

[self.view addSubview:self.playerController.view];

[self.playerController play];
MPMoviePlayerViewController播放本地音频

self.playerViewController= [[MPMoviePlayerViewController alloc]initWithContentURL:url];

[self presentMoviePlayerViewControllerAnimated:self.playerViewController];
```

### MPMoviePlayerViewController播放远程音频

```
NSURL*url = [NSURLURLWithString:@"http://ugc.cdn.qianqian.com/yinyueren/audio/eae34562553f5c48c573d42f281b4b70.mp3"];

self.playerViewController= [[MPMoviePlayerViewController alloc]initWithContentURL:url];

[self presentMoviePlayerViewControllerAnimated:self.playerViewController];

```
