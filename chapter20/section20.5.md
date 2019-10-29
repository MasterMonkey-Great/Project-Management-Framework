# 20.4 iOS 音视频开发-AVPlayer播放本地、远程视频

AVPlayer 属于 AVFoundation 框架，使用时需要先导入``<AVFoundation/AVFoundation.h>``框架头文件。

## AVPlayer 使用

在开发中，单纯使用AVPlayer类是无法显示视频的，要将视频层添加至AVPlayerLayer中，这样才能将视频显示出来。

#### 1、先将在线视频链接存放在NSURL中，然后初始化AVPlayerItem对象，AVPlayerItem是管理资源的对象。

#### 2、然后监听AVPlayerItem的 status 和 loadedTimeRange 属性，status 属性有三种状态：

```
typedef NS_ENUM(NSInteger, AVPlayerItemStatus) {

   AVPlayerItemStatusUnknown,

   AVPlayerItemStatusReadyToPlay,

   AVPlayerItemStatusFailed

};


1.当 AVPlayerItem 的 status 等于AVPlayerStatusReadyToPlay时代表视频已经可以播放了，我们就可以调用play方法播放了。

2.loadedTimeRange 属性代表已经缓冲的进度，监听此属性可以在UI中更新缓冲进度。
```

#### 3、addPeriodicTimeObserverForInterval给AVPlayer 添加time Observer 有利于我们去检测播放进度。

#### 4、使用 AVPlayer 对象的 seekToTime 方法 可以控制视频播放进度。

## 步骤

### 第一步：初始化 播放器对象 以及 播放器层。

```
- (void)initPlayer{

/ /创建播放器层

AVPlayerLayer*playerLayer = [AVPlayerLayer playerLayerWithPlayer:self.player];

playerLayer.frame = self.view.bounds;

[self.view.layer addSublayer:playerLayer];

}
```
#### //初始化播放器

```
-(AVPlayer*)player{

if(!_player) {

NSURL*url = [NSURL URLWithString:@"http://images.apple.com/media/cn/apple-events/2016/5102cb6c_73fd_4209_960a_6201fdb29e6e/keynote/apple-event-keynote-tft-cn-20160908_1536x640h.mp4"];

AVPlayerItem*playerItem=[AVPlayerItem playerItemWithURL:url];

_player= [AVPlayer playerWithPlayerItem:playerItem];

// 通过 KVO 监听AVPlayerItem的status和loadedTimeRange属性

[self addObserverToPlayerItem:playerItem];

// 给播放器添加进度更新 addPeriodicTimeObserverForInterval

[self addProgressObserver];

//给AVPlayerItem添加播放完成通知

[self addNotification];

}

return_player;

}
```

### 第二步：通过 KVO 监听AVPlayerItem的status和loadedTimeRange属性

```
-(void)addObserverToPlayerItem:(AVPlayerItem*)playerItem {

//监控状态属性，注意AVPlayer也有一个status属性，通过监控它的status也可以获得播放状态

[playerItem addObserver:self forKeyPath:@"status" options:NSKeyValueObservingOptionNew context:nil];

//监控网络加载情况属性

[playerItem addObserver:self forKeyPath:@"loadedTimeRanges" options:NSKeyValueObservingOptionNew context:nil];

}
```
```
-(void)observeValueForKeyPath:(NSString*)keyPath ofObject:(id)object change:(NSDictionary*)change context:(void*)context {

AVPlayerItem*playerItem=object;

if([keyPath isEqualToString:@"status"]) {

AVPlayerStatus status= [[change objectForKey:@"new"]intValue];

if(status==AVPlayerStatusReadyToPlay){

NSLog(@"正在播放...，视频总长度:%.2f",CMTimeGetSeconds(playerItem.duration));

}

} else if ([keyPath isEqualToString:@"loadedTimeRanges"]){

NSArray*array=playerItem.loadedTimeRanges;

CMTimeRange timeRange = [array.firstObjectCMTimeRangeValue];//本次缓冲时间范围

float startSeconds =CMTimeGetSeconds(timeRange.start);

float durationSeconds =CMTimeGetSeconds(timeRange.duration);

NSTimeIntervaltotalBuffer = startSeconds + durationSeconds;//缓冲总长度

NSLog(@"共缓冲：%.2f",totalBuffer);

}

}

```

### 第三步：给播放器添加进度更新 addPeriodicTimeObserverForInterval

```
-(void)addProgressObserver{

AVPlayerItem*playerItem=self.player.currentItem;

UIProgressView*progress=self.progress;

//这里设置每秒执行一次

[self.player addPeriodicTimeObserverForInterval:CMTimeMake(1.0,1.0)queue:dispatch_get_main_queue()usingBlock:^(CMTimetime) {

float current=CMTimeGetSeconds(time);

float total=CMTimeGetSeconds([playerItem duration]);

NSLog(@"当前已经播放%.2fs.",current);

];

}
```

### 第四步：给AVPlayerItem添加播放完成通知

```
- (void)addNotification {

[[NSNotificationCenter defaultCenter]addObserver:self

selector:@selector(playbackFinished:)

name:AVPlayerItemDidPlayToEndTimeNotification

object:self.player.currentItem];

}
//播放完成通知

-(void)playbackFinished:(NSNotification*)notification{

   NSLog(@"视频播放完成.");

}

```

### 第五步：播放、暂停视频

```
[self.player play];//播放视频

[self.player pause];//暂停视频

```
以上五步就是播放视频的基本步骤，如果上面的代码让你看的头疼，你也可以去下载文中的代码，根据Demo效果一步一步去看这些方法（Demo中的代码更复杂一些）。

注意：如果你是将文中的代码拷贝到你新建的工程，记得在 Info.plist 允许工程支持HTTP协议（因为文中演示视频是HTTP协议视频）



