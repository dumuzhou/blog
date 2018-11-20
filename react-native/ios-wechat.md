# react-native pod 添加微信sdk

## 添加依赖
``` bash
yarn add react-native-wechat
```

## 进入ios并修改Podfile文件
> React 后添加 `RCTLinkingIOS` 依赖
```
pod 'RCTWeChat',
:path => "../node_modules/react-native-wechat/ios"
```
安装
``` bash
pod install
```

## 打开.xcodespace修改
pods > RCTWeChat > Link Binary With
XCTest.framework 改为 Optional

## 添加虚拟机或真机XCTEST
/Applications//Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/Library/Frameworks/XCTest.framework
XCTest.framework 到 framework 选copy
拖到 Gen面板 > emb

## 修改xcode设置
- build set > bitcode 改为No
- info > URLtype 增加 weixin 微信appid
- info >  LSApplicationQueriesSchemes 添加数组 weixin wechat

## 编辑 AppDelegate.m
头部添加
``` objc
#import <React/RCTLinkingManager.h>
```
添加回调
``` objc
// ios 8.x or older
- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url
sourceApplication:(NSString *)sourceApplication annotation:(id)annotation
{
  return [RCTLinkingManager application:application openURL:url
                            sourceApplication:sourceApplication annotation:annotation];
}

// ios 9.0+
- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url
            options:(NSDictionary<NSString*, id> *)options
{
  return [RCTLinkingManager application:application openURL:url options:options];
}
```

## 测试
``` js
import React, { Component } from 'react';
import { StyleSheet, Text, View, TouchableHighlight, } from 'react-native';
var WeChat=require('react-native-wechat');

class CustomButton extends Component {
  render() {
    return (
      <TouchableHighlight
        style={styles.button}
        underlayColor="#a5a5a5"
        onPress={this.props.onPress}>
        <Text style={styles.buttonText}>{this.props.text}</Text>
      </TouchableHighlight>
    );
  }
}
export default class RNWeChatDemo extends Component {
  constructor(props) {
    super(props);
    //应用注册
    WeChat.registerApp('微信appid');
    WeChat.addListener(
      'SendMessageToWX.Resp',
      (response) => {
        if (parseInt(response.errCode) === 0) {
          console.log('分享成功');
          console.log(response)
        } else {
          console.log('分享失败');
        }
      }
    );
    WeChat.addListener(
      'SendAuth.Resp',
      (response) => {
        if (parseInt(response.errCode) === 0) {
          console.log('登录成功');
          console.log(response)
        } else {
          console.log('登录失败');
        }
      }
    );
    WeChat.addListener(
      'PayReq.Resp',
      (response) => {
        if (parseInt(response.errCode) === 0) {
          console.log('支付成功');
          console.log(response)
        } else {
          console.log('支付失败');
        }
      }
    );
  }
  render() {
    return (
      <View style={{margin:20}}>
        <Text style={styles.welcome}>
          微信好友/朋友圈分享实例
        </Text>
        <CustomButton text='微信好友分享-文本'
          onPress={() => {
            WeChat.isWXAppInstalled()
              .then((isInstalled) => {
                if (isInstalled) {
                  WeChat.shareToSession({type: 'text', description: '测试微信好友分享文本'})
                    .catch((error) => {
                      console.log(error.message);
                    });
                } else {
                  console.log('没有安装微信软件，请您安装微信之后再试');
                }
              });
          }}
        />
        <CustomButton text='微信好友分享-链接'
          onPress={() => {
            WeChat.isWXAppInstalled()
              .then((isInstalled) => {
                if (isInstalled) {
                  WeChat.shareToSession({
                    title:'微信好友测试链接',
                    description: '分享描述',
                    thumbImage: 'https://image.shidexian.com/Public/images1/down-app.png',
                    type: 'news',
                    webpageUrl: 'https://github.com/dumuzhou/blog'
                  })
                    .catch((error) => {
                      console.log(error.message);
                    });
                } else {
                  console.log('没有安装微信软件，请您安装微信之后再试');
                }
              });
          }}
        />
        <CustomButton text='微信朋友圈分享-文本'
          onPress={() => {
            WeChat.isWXAppInstalled()
              .then((isInstalled) => {
                if (isInstalled) {
                  WeChat.shareToTimeline({type: 'text', description: '测试微信朋友圈分享文本'})
                    .catch((error) => {
                      console.log(error.message);
                    });
                } else {
                  console.log('没有安装微信软件，请您安装微信之后再试');
                }
              });
          }}
        />
        <CustomButton text='微信朋友圈分享-链接'
          onPress={() => {
            WeChat.isWXAppInstalled()
              .then((isInstalled) => {
                if (isInstalled) {
                  WeChat.shareToTimeline({
                    title:'微信朋友圈测试链接',
                    description: '分享描述',
                    thumbImage: 'https://image.shidexian.com/Public/images1/down-app.png',
                    type: 'news',
                    webpageUrl: 'https://github.com/dumuzhou/blog'
                  })
                    .catch((error) => {
                      console.log(error.message);
                    });
                } else {
                  console.log('没有安装微信软件，请您安装微信之后再试');
                }
              });
          }}
        />
        <CustomButton text='微信登录'
          onPress={() => {
            let scope = 'snsapi_userinfo';
            let state = 'wechat_sdk_demo';
            WeChat.isWXAppInstalled()
              .then((isInstalled) => {
                if (isInstalled) {
                  WeChat.sendAuthRequest(scope, state)
                    .then(responseCode => {
                      console.log('登录回调')
                      alert('登录回调')
                      console.log(responseCode);
                    })
                    .catch(err => {
                      console.log('登录授权发生错误')
                      alert('登录授权发生错误')
                    })
                } else {
                  console.log('请先安装微信')
                  alert('请先安装微信')
                }
              })
          }}
        />
        <CustomButton text='微信支付'
          onPress={() => {
            WeChat.isWXAppInstalled()
              .then((isInstalled) => {
                if (isInstalled) {
                  WeChat.pay(
                    {
                    // 支付信息
                    }
                  )
                } else {
                  console.log('请先安装微信')
                }
              })
          }}
        />
      </View>
    );
  }
}

const styles = StyleSheet.create({
  welcome: {
    fontSize: 20,
    textAlign: 'center',
    margin: 10,
  },
  button: {
    margin:5,
    backgroundColor: 'white',
    padding: 15,
    borderBottomWidth: StyleSheet.hairlineWidth,
    borderBottomColor: '#cdcdcd',
  },
});

```
