# react-native android 接入支付宝支付

## 引入sdk

下载[SDK](https://docs.open.alipay.com/54/104509)

mkdir android/app/libs

把下载下来的jar文件重命名为libs/alipaySdk.jar并复制到android/app/libs/

vim android/app/build.gradle

```
dependencies {
 ......
 compile files('libs/alipaySdk.jar')
 ......
}
```
vim $(find ./android/app/src -name AndroidManifest.xml)
``` xml
<!-- 支付宝权限声明 -->
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.READ_PHONE_STATE" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />


<!-- 支付宝 activity 声明 -->
<activity
 android:name="com.alipay.sdk.app.H5PayActivity"
 android:configChanges="orientation|keyboardHidden|navigation|screenSize"
 android:exported="false"
 android:screenOrientation="behind"
 android:windowSoftInputMode="adjustResize|stateHidden" >
</activity>
<activity
 android:name="com.alipay.sdk.app.H5AuthActivity"
 android:configChanges="orientation|keyboardHidden|navigation"
 android:exported="false"
 android:screenOrientation="behind"
 android:windowSoftInputMode="adjustResize|stateHidden" >
</activity>
```
vim android/app/proguard-rules.pro
``` java
-keep class com.alipay.android.app.IAlixPay{*;}
-keep class com.alipay.android.app.IAlixPay$Stub{*;}
-keep class com.alipay.android.app.IRemoteServiceCallback{*;}
-keep class com.alipay.android.app.IRemoteServiceCallback$Stub{*;}
-keep class com.alipay.sdk.app.PayTask{ public *;}
-keep class com.alipay.sdk.app.AuthTask{ public *;}
-keep class com.alipay.sdk.app.H5PayCallback {
 <fields>;
 <methods>;
}
-keep class com.alipay.android.phone.mrpc.core.** { *; }
-keep class com.alipay.apmobilesecuritysdk.** { *; }
-keep class com.alipay.mobile.framework.service.annotation.** { *; }
-keep class com.alipay.mobilesecuritysdk.face.** { *; }
-keep class com.alipay.tscenter.biz.rpc.** { *; }
-keep class org.json.alipay.** { *; }
-keep class com.alipay.tscenter.** { *; }
-keep class com.ta.utdid2.** { *;}
-keep class com.ut.device.** { *;}
```
find ./android -name MainApplication.java

在MainApplication.java所在文件夹创建名为alipay的包

vim alipay/AlipayModule.java
``` java
package host.exp.exponent.alipay;

import com.alipay.sdk.app.PayTask;
import com.facebook.react.bridge.Arguments;
import com.facebook.react.bridge.Promise;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.bridge.ReactContextBaseJavaModule;
import com.facebook.react.bridge.ReactMethod;
import com.facebook.react.bridge.WritableMap;
import java.util.Map;

public class AlipayModule extends ReactContextBaseJavaModule {

 public AlipayModule(ReactApplicationContext reactContext) {
   super(reactContext);
 }

 @Override
 public String getName() {
   return "Alipay";
 }

 @ReactMethod
 public void pay(final String orderInfo, final Promise promise) {
   Runnable payRunnable = new Runnable() {
     @Override
     public void run() {
       WritableMap map = Arguments.createMap();
       PayTask alipay = new PayTask(getCurrentActivity());
       Map<String, String> result = alipay.payV2(orderInfo,true);
       for (Map.Entry<String, String> entry: result.entrySet())
         map.putString(entry.getKey(), entry.getValue());
       promise.resolve(map);
     }
   };
   // 必须异步调用
   Thread payThread = new Thread(payRunnable);
   payThread.start();
 }

}
```
vim alipay/AlipayPackage.java
``` java
package host.exp.exponent.alipay;

import com.facebook.react.ReactPackage;
import com.facebook.react.bridge.NativeModule;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.uimanager.ViewManager;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class AlipayPackage implements ReactPackage {

 @Override
 public List<ViewManager> createViewManagers(ReactApplicationContext reactContext) {
   return Collections.emptyList();
 }

 @Override
 public List<NativeModule> createNativeModules(
         ReactApplicationContext reactContext) {
   List<NativeModule> modules = new ArrayList<>();
   modules.add(new AlipayModule(reactContext));
   return modules;
 }

}
```
vim $(find ./android -name MainApplication.java)
``` java
import host.exp.exponent.alipay.AlipayPackage; // 头部添加


@Override
protected List<ReactPackage> getPackages() {
 return Arrays.<ReactPackage>asList(
     new MainReactPackage(),
     new AlipayPackage() // <-- 注册模块
 );
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

