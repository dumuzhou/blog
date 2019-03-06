# EXPO react-native接入阿里云推送

 确认ios pod依赖为2.10.4及以上
> :tag => "ios/2.10.4",


阿里云后台创建app和上传推送证书
[参考](https://help.aliyun.com/document_detail/30071.html?spm=a2c4g.11186623.2.16.40b95b0fWPzAgQ)

## Build Phases > Link Binary With Libraries添加依赖
- libz.tbd 
- libresolv.tbd 
- CoreTelephony.framework 
- SystemConfiguration.framework 
- libsqlite3.tbd 

## 阿里云后台下载AliyunEmasServices-Info.plist 和ios 推送sdk
- AliyunEmasServices-Info.plist文件拖入对应App Target，在弹出框勾选Copy items if needed
- 把下载SDK目录中的framework拖入对应Target，在弹出框勾选Copy items if needed

## Pod依赖

编辑 ios/Podfile
``` 
source 'https://github.com/aliyun/aliyun-specs.git'
...
pod 'AlicloudPush', '~> 1.9.8'
```
pod install

## 根目录新建目录push
push目录下新建CalendarManager.h 和 CalendarManager.m
编辑 CalendarManager.h
``` objc
#import <React/RCTBridgeModule.h>
#import <React/RCTLog.h>

@interface CalendarManager : NSObject <RCTBridgeModule>
+(void) changePush:(NSDictionary *) noti;
+(void) changeToken:(NSString *) str;
@end
```
编辑 CalendarManager.m
``` objc
#import "CalendarManager.h"
@implementation CalendarManager
NSString *token = @"";
NSDictionary *push;

RCT_EXPORT_MODULE();

RCT_EXPORT_METHOD(addEvent:(NSString *)name location:(NSString *)location)
{
    token = @"";
    RCTLogInfo(@"Pretending to crkokokoeate an event %@ at %@", name, location);
}
RCT_EXPORT_METHOD(getPush:(RCTResponseSenderBlock)callback)
{
    NSArray *events = @[push];
    callback(@[[NSNull null], events]);
}
RCT_EXPORT_METHOD(getToken:(RCTResponseSenderBlock)callback)
{
    NSArray *events = @[token];
    callback(@[[NSNull null], events]);
}
+(void) changePush: (NSDictionary *) noti {
    push = noti;
    NSLog(@"waitis %@", push);
}
+(void) changeToken: (NSString *) str {
    token = str;
    NSLog(@"waitis %@", token);
}
@end

```
## 修改AppDelegate.m
头部添加
``` objc
#import <CloudPushSDK/CloudPushSDK.h>
#import "CalendarManager.h"
```
@end前添加 记得修改asyncInit和appSecret 在阿里云后台创建的app查看
``` objc
// push
- (void)initCloudPush {
    // SDK初始化
    [CloudPushSDK asyncInit:@"阿里云asyncInit" appSecret:@"阿里云appSecret" callback:^(CloudPushCallbackResult *res) {
        
        if (res.success) {
            
            [CalendarManager changeToken: [CloudPushSDK getDeviceId]];
            NSLog(@"Push SDK init success, deviceId: %@.", [CloudPushSDK getDeviceId]);
        } else {
            NSLog(@"Push SDK init failed, error: %@", res.error);
        }
    }];
}
/**
 *    注册苹果推送，获取deviceToken用于推送
 *
 *    @param     application
 */
- (void)registerAPNS:(UIApplication *)application {
    if ([[[UIDevice currentDevice] systemVersion] floatValue] >= 8.0) {
        // iOS 8 Notifications
        [application registerUserNotificationSettings:
         [UIUserNotificationSettings settingsForTypes:
          (UIUserNotificationTypeSound | UIUserNotificationTypeAlert | UIUserNotificationTypeBadge)
                                           categories:nil]];
        [application registerForRemoteNotifications];
    }
    else {
        // iOS < 8 Notifications
        [[UIApplication sharedApplication] registerForRemoteNotificationTypes:
         (UIRemoteNotificationTypeAlert | UIRemoteNotificationTypeBadge | UIRemoteNotificationTypeSound)];
    }
}
/*
 *  苹果推送注册成功回调，将苹果返回的deviceToken上传到CloudPush服务器
 */
- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {
    [CloudPushSDK registerDevice:deviceToken withCallback:^(CloudPushCallbackResult *res) {
        if (res.success) {
            NSLog(@"Register deviceToken success.");
        } else {
            NSLog(@"Register deviceToken failed, error: %@", res.error);
        }
    }];
}
/*
 *  苹果推送注册失败回调
 */
- (void)application:(UIApplication *)application didFailToRegisterForRemoteNotificationsWithError:(NSError *)error {
    NSLog(@"didFailToRegisterForRemoteNotificationsWithError %@", error);
}

/**
 *    注册推送消息到来监听
 */
- (void)registerMessageReceive {
    [[NSNotificationCenter defaultCenter] addObserver:self
                                             selector:@selector(onMessageReceived:)
                                                 name:@"CCPDidReceiveMessageNotification"
                                               object:nil];
}
/**
 *    处理到来推送消息
 *
 *    @param     notification
 */
- (void)onMessageReceived:(NSNotification *)notification {
    NSLog(@"收到信息了1");
    CCPSysMessage *message = [notification object];
    NSString *title = [[NSString alloc] initWithData:message.title encoding:NSUTF8StringEncoding];
    NSString *body = [[NSString alloc] initWithData:message.body encoding:NSUTF8StringEncoding];
    //[CalendarManager change: @"ddddddd1"];
    NSLog(@"Receive message title: %@, content: %@.", title, body);
}
/*
 *  App处于启动状态时，通知打开回调
 */
- (void)application:(UIApplication*)application didReceiveRemoteNotification:(NSDictionary*)userInfo {
    NSLog(@"收到信息了");
    // [CalendarManager change: @"ddddddd2"];
    NSLog(@"Receive one notification.");
    // 取得APNS通知内容
    NSDictionary *aps = [userInfo valueForKey:@"aps"];
    // 内容
    NSString *content = [aps valueForKey:@"alert"];
    // badge数量
    NSInteger badge = [[aps valueForKey:@"badge"] integerValue];
    // 播放声音
    NSString *sound = [aps valueForKey:@"sound"];
    // 取得Extras字段内容
    NSString *Extras = [userInfo valueForKey:@"Extras"]; //服务端中Extras字段，key是自己定义的
    NSLog(@"content = [%@], badge = [%ld], sound = [%@], Extras = [%@]", content, (long)badge, sound, Extras);
    // iOS badge 清0
    application.applicationIconBadgeNumber = 0;
    // 通知打开回执上报
    // [CloudPushSDK handleReceiveRemoteNotification:userInfo];(Deprecated from v1.8.1)
    [CloudPushSDK sendNotificationAck:userInfo];
}
```
启动
``` objc
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
//
    NSDictionary *noti=[launchOptions objectForKey:UIApplicationLaunchOptionsRemoteNotificationKey];
    [CalendarManager changePush: @{@"noPush": @"1"}];
    if (noti) {
        [CalendarManager changePush: noti];
    }
    [self registerAPNS:application];
    [self initCloudPush];
    [self registerMessageReceive];
    [CloudPushSDK sendNotificationAck:launchOptions];
    //
  return [super application:application didFinishLaunchingWithOptions:launchOptions];
}
```
## 修改expo推送默认接收的字段
全局搜索 notificationBody 点击匹配到的第一个
``` objc
notificationBody:payload[@"body"]
改为
notificationBody:payload
```
## js接收推送
头部
``` javascript
import { NativeModules } from "react-native";
import { Permissions, Notifications } from "expo";
```
获取推送token和推送消息
``` javascript
  initPush() {
    let CalendarManager = NativeModules.CalendarManager;
    CalendarManager.getToken((error: any, events: any) => {
      if (!error) {
        if (events.length && events[0]) {
          let token = events[0];
          console.log("token");
          console.log(token);
        }
      }
    });
    CalendarManager.getPush((error: any, events: any) => {
      if (!error) {
        if (events.length && !events[0].noPush) {
          console.log("应用未启动时推送");
          let noti = events[0];
          console.log(noti);
        }
      }
    });
    Notifications.addListener((noti: any) => {
      if (noti.origin === "selected" && noti.remote) {
        console.log("应用挂起时推送");
        console.log(this);
        console.log(noti);
      }
    });
  }

```
## 测试 xcode打开推送开关，勾选Background Modes 推送
在真机上运行，打开调试模式，查看log，token就是app对应机器的唯一id,和当前用户关联，需要对特定用户推送消息的话就需要保存，开发模式测试推送也要用到
获取到token之后就是登陆阿里云后台测试推送了，app在挂起和杀死的情况下能收到推送就没问题了。
