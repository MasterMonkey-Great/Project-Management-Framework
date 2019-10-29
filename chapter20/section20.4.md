# 20.4 iOS 音视频开发-AVPlayer播放本地、远程音频

AVAudioPlayer、AVPlayer 属于 AVFoundation 框架，使用时需要先导入 <AVFoundation/AVFoundation.h> 框架头文件。



## AVAudioPlayer

AVAudioPlayer 只能播放本地文件，X流式媒体。

### 导入头文件

```
#import <AVFoundation/AVFoundation.h>

```

### 声明播放器对象属性

```
@property (strong, nonatomic) AVAudioPlayer *player;

```

### AVAudioPlayer 播放本地音频
```
NSError*playerError;

NSURL *url = [[NSBundle mainBundle] URLForResource:@"yxqc.mp3" withExtension:nil];

self.player = [[AVAudioPlayer alloc] initWithContentsOfURL:url error:&playerError];

[self.player setNumberOfLoops:0];// 设置播放循环次数

[self.player setVolume:1];// 音量，0-1之间

[self.player setDelegate:self];

[self.player prepareToPlay];// 分配播放所需的资源，并将其加入内部播放队列

[self.player play];// 开始播放
```

### 代理方法
```
#pragma mark - AVAudioPlayerDelegate

- (void)audioPlayerDidFinishPlaying:(AVAudioPlayer*)player successfully:(BOOL)flag {

  NSLog(@"--- 播放结束 ---");

}
```


## AVPlayer

与AVAudioPlayer不同之处，可以播放远程音乐。可以通过替换item来替换播放文件(而不用通过创建新的player)

### 导入头文件

```
#import <AVFoundation/AVFoundation.h>
```

### 声明播放器对象属性

```
@property (strong, nonatomic) AVPlayer *player;//播放器对象

```

### AVPlayer 播放本地音频
```
NSURL *url = [[NSBundle mainBundle] URLForResource:@"yxqc.mp3" withExtension:nil];

AVPlayerItem *item = [AVPlayerItem playerItemWithURL:url];// 2.创建AVPlayerItem

self.player = [AVPlayer playerWithPlayerItem:item];// 3.创建AVPlayer

[self.player play];

```

### AVPlayer 播放远程音频

```
NSURL *url = [NSURL URLWithString:@"http://ugc.cdn.qianqian.com/yinyueren/audio/eae34562553f5c48c573d42f281b4b70.mp3"];

AVPlayerItem *item = [AVPlayerItem playerItemWithURL:url];// 2.创建AVPlayerItem

self.player = [AVPlayer playerWithPlayerItem:item];// 3.创建AVPlayer

[self.player play];

```

