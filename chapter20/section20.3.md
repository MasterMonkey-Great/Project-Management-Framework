# 20.3 iOS 音视频开发-MediaPlayer播放本地、远程视频

使用 MediaPlayer 时需要先导入<MediaPlayer/MediaPlayer.h> 框架头文件。

MediaPlayer 框架是 iOS 平台上一个用于音频和视频播放的高层级接口，它包含了一个你可以在应用中直接使用的默认的用户界面。你可以使用它来播放用户在 iPod 库中的项目，或者播放本地文件以及网络流。

另外，这个框架也包括了查找用户媒体库中内容的 API，同时还可以配置像是在锁屏界面或者控制中心里的音频控件。



### 导入头文件

```
#import <MediaPlayer/MediaPlayer.h>

```

### 声明播放器对象属性

```
@property(strong, nonatomic)MPMoviePlayerController*playerController;

@property(strong, nonatomic)MPMoviePlayerViewController*playerViewController;
MPMoviePlayerController播放本地视频

// 1.获取视频的URL

// 播放前，更改为本地视频文件

NSString *urlStr= [[NSBundle mainBundle] pathForResource:@"apple.mp4" ofType:nil];

if(urlStr ==nil) {

   NSLog(@"--- 请设置本地视频路径 ---");

}

NSURL*url = [NSURL fileURLWithPath:urlStr];

self.playerController = [[MPMoviePlayerController alloc] initWithContentURL:url];

self.playerController.view.frame = self.view.bounds;

// MPMoviePlayerController 需要添加到 View 上

[self.view addSubview:self.playerController.view];

[self.playerController play];

```


### MPMoviePlayerViewController播放本地视频

```
NSString *urlStr= [[NSBundle mainBundle] pathForResource:@"apple.mp4" ofType:nil];

NSURL*url = [NSURL fileURLWithPath:urlStr];

self.playerViewController = [[MPMoviePlayerViewController alloc] initWithContentURL:url];

[self presentMoviePlayerViewControllerAnimated:self.playerViewController];


```

### MPMoviePlayerViewController播放远程视频

```
// 1.获取视频的URL

NSURL *url = [NSURL URLWithString:@"http://images.apple.com/media/cn/apple-events/2016/5102cb6c_73fd_4209_960a_6201fdb29e6e/keynote/apple-event-keynote-tft-cn-20160908_1536x640h.mp4"];

// 2.创建控制器

playerViewController = [[MPMoviePlayerViewController alloc] initWithContentURL:url];

// 3.播放视频

[self presentMoviePlayerViewControllerAnimated:self.playerViewController];

```

