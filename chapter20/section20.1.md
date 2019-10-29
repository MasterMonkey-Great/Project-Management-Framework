# 20.1 AVAudioRecorder录音、AVAudioPlayer播放音频介绍


AVAudioRecorder、AVAudioPlayer 属于AVFoundation框架，使用时需要先导入<AVFoundation/AVFoundation.h>框架头文件。


## AVFoundation

是苹果的现代媒体框架，它包含了一些不同用途的 API 和不同层级的抽象。其中有一些是Objective-C 对于底层 C 语言接口的封装。除了少数的例外情况，AVFoundation 可以同时在 iOS 和 mac OS  中使用。

## AVAudioRecorder

录音机，提供了在应用程序中的音频记录能力。作为与 AVAudioPlayer 相对应的 API，AVAudioRecorder 是将音频录制为文件的最简单的方法。除了用一个音量计接受音量的峰值和平均值以外，这个 API 简单粗暴，如果你的使用场景很简单的话，这可能恰恰就是你想要的方法。

## AVAudioPlayer

这个高层级的 API 为你提供一个简单的接口，用来播放本地或者内存中的音频。这是一个无界面的音频播放器 (也就是说没有提供 UI 元素)，使用起来也很直接简单。它不适用于网络音频流或者低延迟的实时音频播放。如果这些问题都不需要担心，那么 AVAudioPlayer 可能就是正确的选择。音频播放器的 API 也为我们带来了一些额外的功能，比如循环播放、获取音频的音量强度等等。


[Demo下载](https://github.com/MasterChanMonkey/FF-Media)



## AVAudioRecorder录音使用介绍：

#### 1 导入AVFoundation框架

```
#import<AVFoundation/AVFoundation.h>

```

#### 2 获取沙盒路径

```
- (NSString*)filePath {

NSString*path = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES)lastObject];

NSString*filePath = [path stringByAppendingPathComponent:@"voice.caf"];

returnfilePath;

}


```

#### 3 录音会话设置（小编试了一下，如果不设置录音会话，播放录音的声音会很小）

```
NSError*errorSession =nil;

AVAudioSession* audioSession = [AVAudioSession sharedInstance];//得到AVAudioSession单例对象

[audioSession setCategory:AVAudioSessionCategoryPlayAndRecord error: &errorSession];//设置类别,表示该应用同时支持播放和录音

[audioSession setActive:YES error: &errorSession];//启动音频会话管理,此时会阻断后台音乐的播放.

//设置成扬声器播放

UInt32doChangeDefault =1;

AudioSessionSetProperty(kAudioSessionProperty_OverrideCategoryDefaultToSpeaker, sizeof(doChangeDefault), &doChangeDefault);


```

#### 4 创建录音配置信息的字典

```
NSDictionary*setting =@{

AVFormatIDKey:@(kAudioFormatAppleIMA4),//音频格式

AVSampleRateKey:@44100.0f,//录音采样率(Hz)如：AVSampleRateKey==8000/44100/96000（影响音频的质量）

AVNumberOfChannelsKey:@1,//音频通道数1或2

AVEncoderBitDepthHintKey:@16,//线性音频的位深度8、16、24、32

AVEncoderAudioQualityKey:@(AVAudioQualityHigh)//录音的质量

};


```

#### 5 创建存放录音文件的地址（音频流写入文件的本地文件URL）

```
NSURL*url = [NSURL URLWithString:[self filePath]];

```

#### 6 初始化AVAudioRecorder对象

```
NSError*error;

self.audioRecorder= [[AVAudioRecorder alloc] initWithURL:url settings:setting error:&error];

if(self.audioRecorder) {

   self.audioRecorder.delegate=self;

   self.audioRecorder.meteringEnabled=YES;

   //设置录音时长，超过这个时间后，会暂停单位是秒

   [self.audioRecorder recordForDuration:30];

   //创建一个音频文件，并准备系统进行录制

   [self.audioRecorder prepareToRecord];

} else {

   NSLog(@"Error: %@", [error localizedDescription]);

}


```

#### 7 开始录音

```
[self.audioRecorder record];//开始录音（或者暂停后，继续录音）

[self.audioRecorder pause];//暂停录音

[self.audioRecorder stop];//停止录制并关闭音频文件


```


#### 8 常用 AVAudioRecorderDelegate


```
- (void)audioRecorderDidFinishRecording:(AVAudioRecorder*)recorder successfully:(BOOL)flag;

```


## AVAudioPlayer播放录音文件：



#### 1、初始化AVAudioPlayer对象
```
NSError*error;

self.player= [[AVAudioPlayer alloc] initWithContentsOfURL:[NSURLfile URLWithPath:[self filePath]] error:&error];

//设置播放循环次数

[self.player setNumberOfLoops:0];

[self.player setVolume:1];//音量，0-1之间

//分配播放所需的资源，并将其加入内部播放队列

[self.player setDelegate:self];

[self.player prepareToPlay];
```

####2、播放录音

```
[self.player play];//播放录音

[self.player pause];//暂停播放录音

[self.player stop];//停止播放录音
```

#### 3、常用AVAudioPlayerDelegate

```
- (void)audioPlayerDidFinishPlaying:(AVAudioPlayer*)player successfully:(BOOL)flag;

```

#### 4 AVAudioPlayer播放本地音频文件的代码在Demo中
















[iOS 音视频开发-AVAudioRecorder录音、AVAudioPlayer播放音频介绍（二）](https://www.jianshu.com/p/71d49db93ead)

