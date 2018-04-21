# react-native 安卓端微信支付补充

最近在做react-native的微信内支付，先是按照 [一步一步将微信支付集成到 react-native 应用](https://juejin.im/post/5a33253b51882521033449b2) 这篇文章的步骤进行，iOS可以顺利调起支付页面并完成支付，但是安卓端在支付的时候支付页面一闪而过，直接返回{ errCode: -1 }，google了一下发现，除了[上述文章](https://juejin.im/post/5a33253b51882521033449b2)中的步骤，安卓端还需要额外配置两个地方。

### 1、配置 android/app/src/ 目录下的 AndroidManifest.xml 文件。

要为微信支付新配置一个activity，配置如下。

```xml
<application ...>
    ......
    <!--  微信支付activity -->
    <activity android:name=".wxapi.WXPayEntryActivity"
              android:label="@string/app_name"
              android:exported="true" />
</application>
```

### 2、配置 android/app/ 目录下的 proguard-rules.pro 文件

在这个文件的最后新增下面这行代码：

```pro
-keep class com.tencent.mm.sdk.** { *; }
```

这个文件是配置安卓代码混淆的文件，使用-keep 配置后表示这个类不混淆，在这里我们肯定是不希望混淆微信的android sdk的，所以加上这行，在生产环境打包的情况下安卓微信支付就可以正常调起并回调啦。

注：如果想要在开发打包下测试安卓微信支付，因为在微信端申请appid的时候你填写的签名是应用的release版签名，如果想要在dev模式下测试微信支付，则需要配置下面这个文件，使dev包也使用release包的签名。

### #. 配置a android/app/ 目录下的 build.gradle 文件

找到 signingConfigs 这个配置，改为以下配置

```gradle
// 下面的值根据自己项目的情况更改
signingConfigs {
    release {
        if (project.hasProperty('MYAPP_RELEASE_STORE_FILE')) { // app配置的store file
            storeFile file(MYAPP_RELEASE_STORE_FILE)
            storePassword MYAPP_RELEASE_STORE_PASSWORD // release的store密码
            keyAlias MYAPP_RELEASE_KEY_ALIAS // release的key alias
            keyPassword MYAPP_RELEASE_KEY_PASSWORD // release的key 密码
        }
    }
    debug {
        if (project.hasProperty('MYAPP_RELEASE_STORE_FILE')) {
            storeFile file(MYAPP_RELEASE_STORE_FILE)
            storePassword MYAPP_RELEASE_STORE_PASSWORD
            keyAlias MYAPP_RELEASE_KEY_ALIAS
            keyPassword MYAPP_RELEASE_KEY_PASSWORD
        }
    }
}
```
