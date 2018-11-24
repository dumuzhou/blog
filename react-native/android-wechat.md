# react-native android 添加微信sdk

## 添加依赖
``` bash
yarn add react-native-wechat
```

## 修改配置
vim android/settings.gradle
```
include ':RCTWeChat'
project(':RCTWeChat').projectDir = new File(rootProject.projectDir, '../node_modules/react-native-wechat/android')
```
vim android/app/build.gradle
``` java
dependencies {
  compile project(':RCTWeChat')    // 底部添加
}
```

vim $(find ./android -name MainApplication.java)

``` java
import com.theweflex.react.WeChatPackage;       // 添加头部

@Override
protected List<ReactPackage> getPackages() {
  return Arrays.<ReactPackage>asList(
    new MainReactPackage(), 
    new WeChatPackage()        // 添加这行
  );
}
```
find ./android -name MainApplication.java

在MainApplication.java所在文件夹创建名为wxapi的包

vim wxapi/WXEntryActivity.java

``` java
package 包名.wxapi;

import android.app.Activity;
import android.os.Bundle;

import com.theweflex.react.WeChatModule;

public class WXEntryActivity extends Activity{
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        WeChatModule.handleIntent(getIntent());
        finish();
    }
}
```

vim wxapi/WXPayEntryActivity.java

``` java
package 包名.wxapi;

import android.app.Activity;
import android.os.Bundle;

import com.theweflex.react.WeChatModule;

public class WXPayEntryActivity extends Activity{
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        WeChatModule.handleIntent(getIntent());
        finish();
    }
}
```

vim $(find ./android/app/src -name AndroidManifest.xml)

``` xml
<manifest>
  <application>
    <activity
      android:name="包名.wxapi.WXEntryActivity"
      android:label="@string/app_name"
      android:exported="true"
    />
    <activity
      android:name="包名.wxapi.WXPayEntryActivity"
      android:label="@string/app_name"
      android:exported="true"
    />
  </application>
</manifest>
```

``` bash
vim $(find ./android -name proguard-rules.pro)
```

```
-keep class com.tencent.mm.sdk.** {
   *;
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
