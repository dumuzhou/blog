# react-native支付宝支付

## 引入文件
根目录新建文件夹 Alipay

下载[SDK](https://docs.open.alipay.com/54/104509)并复制到Alipay，倒入

新建 AlipayMoudle.h
``` objc
#import <React/RCTBridgeModule.h>
#import <React/RCTLog.h>

@interface AlipayMoudle : NSObject <RCTBridgeModule>
+(void) handleCallback:(NSURL *)url;
@end
```

新建 AlipayMoudle.m
``` objc
#import "AlipayMoudle.h"
#import <AlipaySDK/AlipaySDK.h>
static RCTPromiseResolveBlock _resolve;
@implementation AlipayMoudle

RCT_EXPORT_METHOD(pay:(NSString *)orderInfo
                  resolver:(RCTPromiseResolveBlock)resolve
                  rejecter:(RCTPromiseRejectBlock)reject){
    //应用注册scheme,在AliSDKDemo-Info.plist定义URL types
    NSString *appScheme = @"alisdkdemo";
    [[AlipaySDK defaultService] payOrder:orderInfo fromScheme:appScheme callback:^(NSDictionary *resultDic) {
        resolve(resultDic);
    }];
    _resolve = resolve;
}

RCT_EXPORT_MODULE(Alipay);
+(void) handleCallback:(NSURL *)url
{
    //如果极简开发包不可用，会跳转支付宝钱包进行支付，需要将支付宝钱包的支付结果回传给开发包
    if ([url.host isEqualToString:@"safepay"]) {
        [[AlipaySDK defaultService] processOrderWithPaymentResult:url standbyCallback:^(NSDictionary *resultDic) {
            //【由于在跳转支付宝客户端支付的过程中，商户app在后台很可能被系统kill了，所以pay接口的callback就会失效，请商户对standbyCallback返回的回调结果进行处理,就是在这个方法里面处理跟callback一样的逻辑】
            _resolve(resultDic);
        }];
    }
    if ([url.host isEqualToString:@"platformapi"]){//支付宝钱包快登授权返回authCode

        [[AlipaySDK defaultService] processAuthResult:url standbyCallback:^(NSDictionary *resultDic) {
            //【由于在跳转支付宝客户端支付的过程中，商户app在后台很可能被系统kill了，所以pay接口的callback就会失效，请商户对standbyCallback返回的回调结果进行处理,就是在这个方法里面处理跟callback一样的逻辑】
            _resolve(resultDic);
        }];
    }
}

@end

```

info 添加  URL alisdkdemo

## 添加依赖
- libc++.tbd
- libz.tbd
- CFNetwork.framework
- CoreGraphics.framework
- CoreMotion.framework
- CoreText.framework
- Foundation.framework
- QuartzCore.framework
- UIKit.framework
- SystemConfiguration.framework
- CoreTelephony.framework

## 修改AppDelegate.m
头部
``` objc
#import <AlipaySDK/AlipaySDK.h>
#import "AlipayMoudle.h"
```
回调
``` objc
// ios 8.x or older
- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url
sourceApplication:(NSString *)sourceApplication annotation:(id)annotation
{
  [AlipayMoudle handleCallback:url];
  return YES;
}

// ios 9.0+
- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url
options:(NSDictionary<NSString*, id> *)options
{
  [AlipayMoudle handleCallback:url];
  return YES;
}
```

## 测试

头部
``` javascript
import { NativeModules } from 'react-native';
var Alipay = NativeModules.Alipay
```

发起支付
``` javascript
Alipay.pay('支付宝支付信息').then(msg=> {
if (msg.resultStatus === '9000') {
  alert('支付成功')
} else {
  alert('支付失败')
}
}, msg=> {
  console.log(msg)
})
```

