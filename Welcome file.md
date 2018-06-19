# WKMPlayer 文档

本文档面向开发人员、测试人员、合作伙伴及对此感兴趣的其他用户，使用该 SDK 需具备基本的 IOS 开发能力。


# 1 功能特性

 - H.264硬编码 
 - AAC 音频硬编码  
 - 弱网丢帧策略  
 - 支持静音操作
#  2 开发准备
## 2.1 开发环境配置
 - Xcode 
 - 更新 WESDK
 - 导入 W3MUI.framework 动态库

## 2.2 设备 & 系统要求

 - 最低支持 iOS 版本：播放端 iOS8.0 
 - 最低支持 iPhone 型号：播放端 iPhone5 
 - 支持 CPU 架构：armv7,arm64,i386,x86_64
 - 含有 i386和 x86_64模拟器版本的库文件，播放功能完全支持模拟器。
# 3 快速集成
下载 [W3MUI.framework.zip](https://onebox.huawei.com/p/b95819b787020a3ae4ad1f67260c777f) W3MUI.framework.zip 解压缩到本地目录，或者更新 WESDK 库。
>最终开发完成发布后才可以在 WESDK 基础库中更新到包含了 WKMPlayer 的W3MUI.framework

下载 [ WKMPlayerDemo.zip](https://onebox.huawei.com/p/7d4e4af26d6f0da62b769999b26b9c21)解压到本地目录
目录结构：

 - Embedded 依赖库嵌入目录
 - W3MUIDemo 怎么使用 WKMPlayer 的 Demo 示例代码
 - WEPlus 主工程
 - WESDK 基础依赖库（包括了 W3MUI.framework）
 - WE.xcworkspace 工作空间（双击打开如下图）
 2图
 
## 3.1 基础集成

#### 3.1.1 导入动态库
将 W3MUI.framework 引入工程。
按照图片中序号设置好 W3MUI.framework 库作为您的工程依赖。
1图

3图

库文件说明
|类文件|说明  |
|--|--|
| WKMPlayerDelegate.h | 这协议是供调用方做回调用的，比如播放进度播放值的变化，播放器状态的变化等等 |
| WKMPlayerProtocol.h|播放器自身的一些能力，具体的播放器实现必须都实现了|
| WKMPlayer.h | 播放器对外公开的方法（接口）使用者可以调用，最终会由具体的播放器提供商实现，调用方无需关心具体实现是 ucloud 还是其他播放器，播放器对外公开的类 |
| WKMStandardPlayer.h | 播放器的标准实现，提供标准的 UI 交互 |
| WKMPlayerViewController.h | 集成了标准播放器的控制器类，提供一个初始化 URL 参数即可使用 |

#### 3.1.2  协议说明
##### 3.1.2.1 WKMPlayerDelegate
这个协议主要包括播放器状态变化相关的回调，比如播放中，暂停，停止，播放错误，以及拖放进度，缓冲值变化等等。

当播放器初始化完毕，加载资源成功可以播放时会回调下面这个代理方法
```c
/**
告诉调用方播放器已经准备好播放，收到这个回调才可以调用 play 方法
@param player 播放器
*/
- (void)didPreparedToPlay:(id<WKMPlayerProtocol>)player;
```
 当播放器回调这个下面代理方法告诉你播放结束了。
```c
/**
告诉调用方委托对象，播放结束了
@param player 播放器
@param reason 结束原因
@param error 错误信息
*/
- (void)player:(id<WKMPlayerProtocol>)player didFinishForReason:(id)reason error:(id)error;
```
 当播放器状态发生变化时，播放器通过下面这个方法告诉你状态信息
  
```c
/**
播放器状态切换回调
@param player 播放器
@param state  当前播放器状态
*/
- (void)player:(id<WKMPlayerProtocol>)player didChangePlayState:(MPMoviePlaybackState)state;
```
关联的状态枚举值如下：

```c
typedef NS_ENUM(NSInteger, MPMoviePlaybackState) {
	MPMoviePlaybackStateStopped,
	MPMoviePlaybackStatePlaying,
	MPMoviePlaybackStatePaused,
	MPMoviePlaybackStateInterrupted,
	MPMoviePlaybackStateSeekingForward,
	MPMoviePlaybackStateSeekingBackward
}
```
播放器流数据加载状态改变时，通过下面方法告诉你
 ```c
 /**
播放时网络加载状态改变
@param player 播放器
@param loadState 加载状态
*/
- (void)player:(id<WKMPlayerProtocol>)player didChangeLoadState:(MPMovieLoadState)loadState;
```
关联枚举值如下：
```c
typedef NS_OPTIONS(NSUInteger, MPMovieLoadState) {
	MPMovieLoadStateUnknown  = 0,
	MPMovieLoadStatePlayable = 1 << 0,
	MPMovieLoadStatePlaythroughOK  = 1 << 1,
	MPMovieLoadStateStalled  = 1 << 2,
}
```
当播放器进度值发生变化时，播放器通过下面回调方法告诉你。
```c
/**
当前播放进度值变化的回调
@param player 播放器
@param time 进度
*/
- (void)player:(id<WKMPlayerProtocol>)player currentPlayTime:(NSTimeInterval)time;
```

播放器进度、缓冲值、可播放时长值、播放总时长变化通过下面这个方法告诉你。
```c
/**
当前播放器缓冲进度值变化的回调
@param player 播放器
@param progress 缓冲进度
@param playableDuration 媒体可播放时长，主要用于表示网络媒体已下载视频时长
@param totalDuration 总长
*/
- (void)player:(id<WKMPlayerProtocol>)player bufferingProgress:(NSInteger)progress playableDuration:(NSTimeInterval)playableDuration totalDuration:(NSTimeInterval)totalDuration;
```
当拖放快进快退成功时，播放器通过这个方法告诉你
```c
/**
快进完成
@param player 播放器
@param time 快进值
*/
- (void)player:(id<WKMPlayerProtocol>)player seekSuccess:(NSNumber *)time;
```
当播放器拖放快进快退失败了，通过这个方法告诉你。
```c
/**
快进失败
@param player 播放器
@param error 错误
*/
- (void)player:(id<WKMPlayerProtocol>)player seekError:(NSNumber *)error;
```

当播放器开始播放并且获取到了第一帧图片数据时，会通过下面这个方法告诉你。
```c
/**
接收到第一帧视频
@param player 播放器
@param didReceived 是否
*/
- (void)player:(id<WKMPlayerProtocol>)player didReceiveFirstVideoFrame:(BOOL)didReceived;
```
当播放器开始播放，且第一帧音频数据传递过来时，通过下面回调方法告诉你。
```c
/**
接收到第一帧音频
@param player 播放器
@param didReceived 是否
*/
- (void)player:(id<WKMPlayerProtocol>)player didReceiveFirstAudioFrame:(BOOL)didReceived;
```


##### 3.1.2.2 WKMPlayerProtocol
播放器自身的一些能力，具体的播放器实现必须都实现了，这个协议是播放器的具体实现类来实现的，播放器开放的标准方法。

下面属性 playerView 所有具体的播放器实现都会拥有此属性，这个就是播放视图
```c
/** 播放显示视图 */
@property (nonatomic ,strong) UIView *playerView;
```
下面方法就是播放器初始化都要实现的构造方法。通过这个方法配置播放器类型，是否支持自动播放，是否支持重复播放的属性。
```c
/**
构建一个播放器
@param delegate 代理委托
@param videoType 视频类型
@param autoPlay 是否自动播放
@param repeat 是否重复播放
@return 播放器实例
*/
- (instancetype)initWithDelegate:(id)delegate videoType:(WKMVideoType)videoType shoudAutoPlay:(BOOL)autoPlay shouldRepeatPlay:(BOOL)repeat;
```

可以传递 URL 给播放器来播放具体内容。通过这个方法会立即初始化播放器实例
```c
/**
初始化播放器
@param url 播放地址
*/
- (void)startPlayWithUrl:(NSString *)url;
```
准备播放（更新播放时间后最好调用该方法）快进快推拖动后继续播放需要调用这个，ios 播放器底层。
```c
/**
准备播放（更新播放时间后最好调用该方法）快进快推拖动后继续播放需要调用这个，ios 播放器底层
*/
- (void)prepareToPlay;
```
播放方法，在播放器初始化以后，可以调用这个方法把暂停状态下的播放器恢复到播放状态。
```c
/**
播放
*/
- (void)play;
```
当播放器在播放中，通过这个方法可以暂停播放内容。
```c
/**
*  暂停
*/
- (void)pause;
```
停止当前播放器的播放内容。
```c
/**
*  停止
*/
- (void)stop;
```
类同与 play 方法，底层调用的就是播放器的 play 方法。让暂停状态下的播放器继续播放。
```c
/**
恢复
*/
- (void)resume;
```
更新播放器进度，一般用于快进快退，设定好具体时间就可以实现快进快退了。
```c
/**
*  更新播放时间
*/
- (void)seekToPlaybackTime:(NSTimeInterval)time;
```
当退出或者不在使用播放器实例的时候，调用这个方法会销毁掉播放器实例。
```c
/**
*  关闭播放器
*/
- (void)shutDown;
```
通过下面这个方法可以主动获取到播放器当前是否在播放的状态。
```c
/**
当前是否播放状态
@return 当前播放状态?
*/
- (BOOL)isPlaying;
```
通过下面这个方法可以主动获取到播放器自身的视图
```c
/**
返回播放器视图
@return 播放器视图
*/
- (UIView *)playBackView;
```
通过下面这个方法可以主动获取到播放器的容器视图
```c
/**
播放器容器视图
@return 视图
*/
- (UIView *)playerView;
```
通过下面这个方法可以主动获取到播放器当前的播放状态。
```c
/**
*  播放器状态
*/
- (MPMoviePlaybackState)playerStatus;
```
通过下面这个方法可以主动获取到当前播放的进度值。
```c
/**
*  当前播放的位置
*/
- (NSTimeInterval)currentPlaybackTime;
```
通过下面这个方法可以主动获取到播放器的视频总时长
```c
/**
*  视频总时间
*/
- (NSTimeInterval)videoDuration;
```
通过下面这个方法可以获取到当前已缓存到本地的视频总时长
```c
/**
*  媒体可播放时长，主要用于表示网络媒体已下载视频时长
*/
- (NSTimeInterval)playableDuration;
```
通过下面这个方法可以获取到缓冲进度。

```c
/**
*  当前缓冲进度
*/
- (NSInteger)bufferingProgress;
```
通过下面这个方法可以主动获取到播放器的状态
```c
/**
视频播放状态
@return 视频状态
*/
- (WKMPlayerState)videoStatus;
```
状态值如下
```c
typedef NS_ENUM(NSInteger, WKMPlayerState) {
	WKMPlayerStateUnknow = 0, 	// 未知
	WKMPlayerStatePlaying  = 1, // 播放中
	WKMPlayerStatePaused = 2, 	// 暂停中
	WKMPlayerStateStoped = 3, 	// 停止了
};
```
通过这个方法可以主动获取到当前播放视频是否取得了第一帧图片。
```c
/**
是否已接受到视频第一帧
@return 是否
*/
- (BOOL)receivedFirstFrame;
```
通过下面方法可以主动获取到自动重连的配置值。
```c
/**
*  设备网络切换时是否自动重连,默认开启(YES)
*/
- (BOOL)reconEnable;
```
通过下面这个方法可以主动获取到播放器是否初始化完整。
```c
/**
* 播放器是否初始化完整
*/
- (BOOL)isActived;
```
通过下面这个方法可以主动获取到播放器最大重连次数的配置值
```c
/**
*  最大重连次数
*/
- (NSUInteger)maxReconCount;
```
通过下面这个方法可以主动获取到播放器直播状态下准备完成的超时时间配置值
```c
/**
暂时适用于直播，播放准备完成超时时间，单位 ms，默认10000，推荐范围5000-150000 不是所有播放器实现都支持，需要适配
@return 播放准备完成超时时间
*/
- (NSInteger)prepareTimeout;
```
通过下面这个方法可以主动获取到播放器当前视频的帧率。
```c
/**
* 当前视频的帧率
*/
- (NSString *)videofps;
```
通过下面这个方法可以主动获取到播放器的码率
```c
/**
* 码率（点播可用）
*/
- (NSString *)biteRate;
```

#### 3.1.3 代码集成
##### 3.1.3.1 引入头文件
```c
#import <W3MUI/W3MUI.h>
```
##### 3.1.3 .2 提供 URL 快速实例化一个标准播放器的控制器
```c
WKMPlayerViewController *vc = [[WKMPlayerViewController alloc] initWithUrl:@“http://abc.com/xyz.mp4”];
[self.navigationController pushViewController:vc animated:YES];
// [self presentViewController:vc animated:YES  completion:nil];
```
> 注意，支持 Push 和 Present，自动处理返回按钮的 Pop 和 Dismiss。    
##### 3.1.3 .3  初始化 WKMStandardPlayer 一个具有标准播放器 UI 交互的视图 UIView 子类
```c
NSString *url = @"http://abc/xyz.mp4";
WKMStandardPlayer *player = [[WKMStandardPlayer alloc] initWithUrl:url];
player.delegate = self;
[self.view addSubview:player];
[player mas_makeConstraints:^(MASConstraintMaker *make) {
	make.edges.equalTo(self.view);
}];
[player startPlay];

// 设置上下左右内容
player.topView.backgroundColor = [[UIColor redColor] colorWithAlphaComponent:.5];
player.leftView.backgroundColor = [[UIColor yellowColor] colorWithAlphaComponent:.5];
player.rightView.backgroundColor = [[UIColor blueColor] colorWithAlphaComponent:.5];
player.centerView.backgroundColor = [[UIColor greenColor] colorWithAlphaComponent:.5];

// 在右边视图添加自定义的一个按钮
UIButton *btn = [UIButton buttonWithType:(UIButtonTypeCustom)];
btn.backgroundColor = [UIColor grayColor];
[btn setTitle:@"TEST" forState:(UIControlStateNormal)];
[player.rightView addSubview:btn];
[btn mas_makeConstraints:^(MASConstraintMaker *make) {
	make.width.height.mas_equalTo(60);
	make.center.equalTo(player.rightView);
}];
```
> 注意，上述代码依赖 Masonry 库。
##### 3.1.3 .4 实例化 WKMPlayer 播放器视图，完全自定义所有 UI 包括进度条。
```c
- (void)viewDidLoad {
	[super viewDidLoad];
	NSString *url = @"http://xyz.com/a.mp4";
	self.player = [[WKMPlayer alloc] initWithDelegate:self videoType:(WKMVideoTypeHttp) shoudAutoPlay:YES shouldRepeatPlay:YES];
	[self.player startPlayWithUrl:url];
	[self.view addSubview:self.player.playerView];
	[self.player.playerView  mas_makeConstraints:^(MASConstraintMaker *make) {
		make.width.equalTo(self.view);
		make.height.equalTo(self.view).multipliedBy(.5); // 占用屏幕一半高度
	}];
	// 增加一个自定义按钮到播放器视图
	UIButton *btn = [UIButton buttonWithType:(UIButtonTypeCustom)];
	[btn setTitle:@"TEST" forState:(UIControlStateNormal)];
	[btn setBackgroundColor:[UIColor redColor]];
	[self.player.playerView  addSubview:btn];
	[btn mas_makeConstraints:^(MASConstraintMaker *make) {
		make.width.height.mas_equalTo(100);
		make.center.equalTo(self.player.playerView);
	}];
}
```
> 注意，上述代码依赖 Masonry 库。

<!--stackedit_data:
eyJoaXN0b3J5IjpbNjc5Mzg4MTQ0XX0=
-->