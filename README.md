
#  iOS 10  适配

随着iOS10已经发布,大家的App都需要适配，下面是总结的一些关于iOS10适配方面的问题

##1.系统判断方法失效

> 在你的项目中，当需要判断系统版本的话,不要使用下面的方法:
> '#define isiOS10([[[[UIDevice currentDevice]systemVersion] substringToIndex:1]intValue >= 10)
> 他会永远返回NO,substringToIndex:1在iOS10会被检测成iOS1了

##应该使用下面的这些方法

>'#define SYSTEM_VERSION_EQUAL_TO(v) ([[[UIDevice currentDevice] systemVersion] compare:v options:NSNumericSearch] == NSOrderedSame)
>
> '#define SYSTEM_VERSION_GREATER_THAN(v) ([[[UIDevice currentDevice] systemVersion] compare:v options:NSNumericSearch] == NSOrderedDescending)

> '#define SYSTEM_VERSION_GREATER_THAN_OR_EQUAL_TO(v) ([[[UIDevice currentDevice] systemVersion] compare:v options:NSNumericSearch] != NSOrderedAscending)

>'#define SYSTEM_VERSION_LESS_THAN(v) ([[[UIDevice currentDevice] systemVersion] compare:v options:NSNumericSearch] == NSOrderedAscending)

>'#define SYSTEM_VERSION_LESS_THAN_OR_EQUAL_TO(v) ([[[UIDevice currentDevice] systemVersion] compare:v options:NSNumericSearch] != NSOrderedDescending)

##或者使用

>if ([[NSProcessInfo processInfo] isOperatingSystemAtLeastVersion:(NSOperatingSystemVersion){.majorVersion = 9, .minorVersion = 1, .patchVersion = 0}]) { NSLog(@"Hello from > iOS 9.1");}
>
>if ([NSProcessInfo.processInfo isOperatingSystemAtLeastVersion:(NSOperatingSystemVersion){9,3,0}]) { NSLog(@"Hello from > iOS 9.3");}


##或者使用
> if (NSFoundationVersionNumber > NSFoundationVersionNumber_iOS_9_0) { // do stuff for iOS 9 and newer} else { // do stuff for older versions than iOS 9}

##2.隐私数据访问问题

你的项目中访问了隐私数据,比如:相机,相册,联系人等,在Xcode8中打开编译的话,统统会crash,控制台会输出下面这样的日志

![](http://upload-images.jianshu.io/upload_images/122816-135f4a89ba4b0ee5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这是因为iOS对用户的安全和隐私的增强,在申请很多私有权限的时候都需要添加描述,但是,在使用Xcode8之前的Xcode还是使用系统的权限通知框.要想解决这个问题,只需要在 info.plist添加NSContactsUsageDescription的key，value自己随意填写就可以了,这里举出对应的key(source code模式下):

![](http://upload-images.jianshu.io/upload_images/2027951-75458e417c912e2a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/2027951-46ce716a15513ca1.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



```
// 相机
<key>NSCameraUsageDescription</key>
<string>App需要您的同意,才能访问相册</string>

// 相册
<key>NSPhotoLibraryUsageDescription</key>
<string>App需要您的同意,才能访问相机</string>

// 麦克风：
<key>NSMicrophoneUsageDescription</key>
<string>App需要您的同意,才能访问麦克风</string>

// 通信录
<key>NSContactsUsageDescription</key>
<string>App需要您的同意,才能访问通信录</string>
```

其它权限配置选项:

```
// 位置
<key>NSLocationUsageDescription</key> 
<string>App需要您的同意,才能访问位置</string> 

// 在使用期间访问位置
<key>NSLocationWhenInUseUsageDescription</key> 
<string>App需要您的同意,才能在使用期间访问位置</string> 

// 始终访问位置
<key>NSLocationAlwaysUsageDescription</key> 
<string>App需要您的同意,才能始终访问位置</string> 

// 日历
<key>NSCalendarsUsageDescription</key> 
<string>App需要您的同意,才能访问日历</string> 

// 提醒事项
<key>NSRemindersUsageDescription</key> 
<string>App需要您的同意,才能访问提醒事项</string> 

// 运动与健身
<key>NSMotionUsageDescription</key>
<string>App需要您的同意,才能访问运动与健身</string> 

// 健康更新
<key>NSHealthUpdateUsageDescription</key> 
<string>App需要您的同意,才能访问健康更新 </string> 

// 健康分享
<key>NSHealthShareUsageDescription</key> 
<string>App需要您的同意,才能访问健康分享</string> 

// 蓝牙
<key>NSBluetoothPeripheralUsageDescription</key> 
<string>App需要您的同意,才能访问蓝牙</string> 

// 媒体资料库
<key>NSAppleMusicUsageDescription</key> 
<string>App需要您的同意,才能访问媒体资料库</string>
```
   
## 3.UIColor的问题

大多数core开头的图形框架和AVFoundation都提高了对扩展像素和宽色域色彩空间的支持.通过图形堆栈扩展这种方式比以往支持广色域的显示设备更加容易。现在对UIKit扩展可以在sRGB的色彩空间下工作，性能更好,也可以在更广泛的色域来搭配sRGB颜色.如果你的项目中是通过低级别的api自己实现图形处理的,建议使用sRGB,也就是说在项目中使用了RGB转化颜色的建议转换为使用sRGB,在UIColor类中新增了两个api:

```
- (UIColor *)initWithDisplayP3Red:(CGFloat)displayP3Red green:(CGFloat)green blue:(CGFloat)blue alpha:(CGFloat)alpha NS_AVAILABLE_IOS(10_0);

+ (UIColor *)colorWithDisplayP3Red:(CGFloat)displayP3Red green:(CGFloat)green blue:(CGFloat)blue alpha:(CGFloat)alpha NS_AVAILABLE_IOS(10_0);
```

## 4.真彩色的显示
真彩色的显示会根据光感应器来自动的调节达到特定环境下显示与性能的平衡效果，如果需要这个功能的话,可以在info.Plist里配置(在source Code模式下):

```
<key>UIWhitePointAdaptivityStyle</key>
```
它有五种取值,分别是:
```
<string>UIWhitePointAdaptivityStyleStandard</string> // 标准模式
<string>UIWhitePointAdaptivityStyleReading</string> // 阅读模式
<string>UIWhitePointAdaptivityStylePhoto</string> // 图片模式
<string>UIWhitePointAdaptivityStyleVideo</string> // 视频模式
<string>UIWhitePointAdaptivityStyleStandard</string> // 游戏模式
```

## 5.在iOS 10 中info.plist文件新加入了NSAllowsArbitraryLoadsInWebContent键

对于网页浏览和视频播放的行为，iOS 10 中新加入了 NSAllowsArbitraryLoadsInWebContent键。通过将它设置为 YES，可以让你的 app 中的 WKWebView 和使用 AVFoundation 播放的在线视频不受 ATS 的限制。这也应该是绝大多数使用了相关特性的 app 的选择。但是坏消息是这个键在 iOS 9 中并不会起作用。

## 6.ATS的问题

1.认情况下你的 app 可以访问加密足够强 (TLS v1.2 以上，AES-128 和 SHA-2 以及 ECDHC 等) 的 HTTPS 内容。这对所有的网络请求都有效，包括 NSURLSession，UIWebView 以及 WKWebView 等。

2.你依然可以添加 NSAllowsArbitraryLoads 为 YES 来禁用 ATS，不过如果你这么做的话，需要在提交 app 时进行说明，为什么需要访问非 HTTPS 内容。一般来说，可能类似浏览器类的 app 比较容易能通过。

3.相比于使用 NSAllowsArbitraryLoads 将全部 HTTP 内容开放，选择使用 NSExceptionDomains 来针对特定的域名开放 HTTP 应该要相对容易过审核。“需要访问的域名是第三方服务器，他们没有进行 HTTPS 对应”会是审核时的一个可选理由，但是这应该只需要针对特定域名，而非全面开放。如果访问的是自己的服务器的话，可能这个理由会无法通过

### 指定域名禁用ATS

- 在info.Plist中配置App的服务域名baidu.com 支持HTTP为例：

```
<key>NSAppTransportSecurity</key>    
  <dict>    
      <key>NSExceptionDomains</key>    
      <dict>    
          <key>baidu.com</key>    
          <dict>    
               <key>NSExceptionAllowsInsecureHTTPLoads</key>//白名单指定域名禁用ATS
               <false/>
               <key>NSIncludesSubdomains</key>
               <true/>
          </dict>    
      </dict>    
  </dict>    
  
```

## 7.UIStatusBar的问题:

在iOS10中,如果还使用以前设置UIStatusBar类型或者控制隐藏还是显示的方法,会报警告,方法过期,如下图:

![](http://upload-images.jianshu.io/upload_images/122816-6cc72fac7695aefa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

上面方法到 iOS 10 不能使用了,要想修改UIStatusBar的样式或者状态使用下图中所示的属性或方法:

```
@property(nonatomic, readonly) UIStatusBarStyle preferredStatusBarStyle NS_AVAILABLE_IOS(7_0) __TVOS_PROHIBITED;
// Defaults to UIStatusBarStyleDefault

@property(nonatomic, readonly) BOOL prefersStatusBarHidden NS_AVAILABLE_IOS(7_0) __TVOS_PROHIBITED; 
// Defaults to NO

- (UIStatusBarStyle)preferredStatusBarStyle NS_AVAILABLE_IOS(7_0) __TVOS_PROHIBITED; 
// Defaults to UIStatusBarStyleDefault
 
- (BOOL)prefersStatusBarHidden NS_AVAILABLE_IOS(7_0) __TVOS_PROHIBITED;
// Defaults to NO

// Override to return the type of animation that should be used for status bar changes for this view controller. This currently only affects changes to prefersStatusBarHidden.
- (UIStatusBarAnimation)preferredStatusBarUpdateAnimation NS_AVAILABLE_IOS(7_0) __TVOS_PROHIBITED;
// Defaults to UIStatusBarAnimationFade
```

## 8.UITextField

在iOS 10 中,UITextField新增了textContentType字段,是UITextContentType类型,它是一个枚举,作用是可以指定输入框的类型,以便系统可以分析出用户的语义.是电话类型就建议一些电话,是地址类型就建议一些地址.可以在```#import <UIKit/UITextInputTraits.h>``` 文件中,查看textContentType字段,有以下可以选择的类型:

```
UIKIT_EXTERN UITextContentType const UITextContentTypeName                      NS_AVAILABLE_IOS(10_0);
UIKIT_EXTERN UITextContentType const UITextContentTypeNamePrefix                NS_AVAILABLE_IOS(10_0);
UIKIT_EXTERN UITextContentType const UITextContentTypeGivenName                 NS_AVAILABLE_IOS(10_0);
UIKIT_EXTERN UITextContentType const UITextContentTypeMiddleName                NS_AVAILABLE_IOS(10_0);
UIKIT_EXTERN UITextContentType const UITextContentTypeFamilyName                NS_AVAILABLE_IOS(10_0);
UIKIT_EXTERN UITextContentType const UITextContentTypeNameSuffix                NS_AVAILABLE_IOS(10_0);
UIKIT_EXTERN UITextContentType const UITextContentTypeNickname                  NS_AVAILABLE_IOS(10_0);
UIKIT_EXTERN UITextContentType const UITextContentTypeJobTitle                  NS_AVAILABLE_IOS(10_0);
UIKIT_EXTERN UITextContentType const UITextContentTypeOrganizationName          NS_AVAILABLE_IOS(10_0);
UIKIT_EXTERN UITextContentType const UITextContentTypeLocation                  NS_AVAILABLE_IOS(10_0);
UIKIT_EXTERN UITextContentType const UITextContentTypeFullStreetAddress         NS_AVAILABLE_IOS(10_0);
UIKIT_EXTERN UITextContentType const UITextContentTypeStreetAddressLine1        NS_AVAILABLE_IOS(10_0);
UIKIT_EXTERN UITextContentType const UITextContentTypeStreetAddressLine2        NS_AVAILABLE_IOS(10_0);
UIKIT_EXTERN UITextContentType const UITextContentTypeAddressCity               NS_AVAILABLE_IOS(10_0);
UIKIT_EXTERN UITextContentType const UITextContentTypeAddressState              NS_AVAILABLE_IOS(10_0);
UIKIT_EXTERN UITextContentType const UITextContentTypeAddressCityAndState       NS_AVAILABLE_IOS(10_0);
UIKIT_EXTERN UITextContentType const UITextContentTypeSublocality               NS_AVAILABLE_IOS(10_0);
UIKIT_EXTERN UITextContentType const UITextContentTypeCountryName               NS_AVAILABLE_IOS(10_0);
UIKIT_EXTERN UITextContentType const UITextContentTypePostalCode                NS_AVAILABLE_IOS(10_0);
UIKIT_EXTERN UITextContentType const UITextContentTypeTelephoneNumber           NS_AVAILABLE_IOS(10_0);
UIKIT_EXTERN UITextContentType const UITextContentTypeEmailAddress              NS_AVAILABLE_IOS(10_0);
UIKIT_EXTERN UITextContentType const UITextContentTypeURL                       NS_AVAILABLE_IOS(10_0);
UIKIT_EXTERN UITextContentType const UITextContentTypeCreditCardNumber          NS_AVAILABLE_IOS(10_0);
```

## 9.UICollectionViewCell的的优化
在iOS 10 之前,UICollectionView上面如果有大量cell,当用户活动很快的时候,整个UICollectionView的卡顿会很明显,为什么会造成这样的问题,这里涉及到了iOS 系统的重用机制,当cell准备加载进屏幕的时候,整个cell都已经加载完成,等待在屏幕外面了,也就是整整一行cell都已经加载完毕,这就是造成卡顿的主要原因,专业术语叫做:掉帧.
要想让用户感觉不到卡顿,我们的app必须帧率达到60帧/秒,也就是说每帧16毫秒要刷新一次.

**iOS 10 之前UICollectionViewCell的生命周期是这样的:**

- 1.用户滑动屏幕,屏幕外有一个cell准备加载进来,把cell从reusr队列拿出来,然后调用prepareForReuse方法,在这个方法里面,可以重置cell的状态,加载新的数据;

- 2.继续滑动,就会调用cellForItemAtIndexPath方法,在这个方法里面给cell赋值模型,然后返回给系统;

- 3.当cell马上进去屏幕的时候,就会调用willDisplayCell方法,在这个方法里面我们还可以修改cell,为进入屏幕做最后的准备工作;

- 4.执行完willDisplayCell方法后,cell就进去屏幕了.当cell完全离开屏幕以后,会调用didEndDisplayingCell方法.


**iOS 10 UICollectionViewCell的生命周期是这样的:**

- 1.用户滑动屏幕,屏幕外有一个cell准备加载进来,把cell从reusr队列拿出来,然后调用prepareForReuse方法,在这里当cell还没有进去屏幕的时候,就已经提前调用这个方法了,对比之前的区别是之前是cell的上边缘马上进去屏幕的时候就会调用该方法,而iOS 10 提前到cell还在屏幕外面的时候就调用;

- 2.在cellForItemAtIndexPath中创建cell，填充数据，刷新状态等操作,相比于之前也提前了;

- 3.用户继续滑动的话,当cell马上就需要显示的时候我们再调用willDisplayCell方法,原则就是:何时需要显示,何时再去调用willDisplayCell方法;

- 4.当cell完全离开屏幕以后,会调用didEndDisplayingCell方法,跟之前一样,cell会进入重用队列.在iOS 10 之前,cell只能从重用队列里面取出,再走一遍生命周期,并调用cellForItemAtIndexPath创建或者生成一个cell.在iOS 10 中,系统会cell保存一段时间,也就是说当用户把cell滑出屏幕以后,如果又滑动回来,cell不用再走一遍生命周期了,只需要调用willDisplayCell方法就可以重新出现在屏幕中了.
iOS 10 中,系统是一个一个加载cell的,二以前是一行一行加载的,这样就可以提升很多性能;

**iOS 10 新增加的Pre-Fetching预加载**

  这个是为了降低UICollectionViewCell在加载的时候所花费的时间,在 iOS 10 中,除了数据源协议和代理协议外,新增加了一个UICollectionViewDataSourcePrefetching协议,这个协议里面定义了两个方法:

```
- (void)collectionView:(UICollectionView *)collectionView prefetchItemsAtIndexPaths:(NSArray<NSIndexPath *> *)indexPaths NS_AVAILABLE_IOS(10_0);

- (void)collectionView:(UICollectionView *)collectionView cancelPrefetchingForItemsAtIndexPaths:(NSArray<NSIndexPath *> *)indexPaths  NS_AVAILABLE_IOS(10_0);
```
在ColletionView prefetchItemsAt indexPaths这个方法是异步预加载数据的,当中的indexPaths数组是有序的,就是item接收数据的顺序;
CollectionView cancelPrefetcingForItemsAt indexPaths这个方法是可选的,可以用来处理在滑动中取消或者降低提前加载数据的优先级.
注意:这个协议并不能代替之前读取数据的方法,仅仅是辅助加载数据.
Pre-Fetching预加载对UITableViewCell同样适用

## 10.UIRefreshControl的使用

在iOS 10 中, UIRefreshControl可以直接在UICollectionView和UITableView中使用,并且脱离了UITableViewController.现在RefreshControl是UIScrollView的一个属性.
使用方法:
```
UIRefreshControl *refreshControl = [[UIRefreshControl alloc] init];
    [refreshControl addTarget:self action:@selector(loadData) forControlEvents:UIControlEventValueChanged];
    collectionView.refreshControl = refreshControl;
```




