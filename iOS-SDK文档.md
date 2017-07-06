## XPaySDK

## 简介
项目地址：[https://github.com/Casum/XPaySDK](https://github.com/Casum/XPaySDK)

lib 文件夹下是 iOS SDK 文件，
example 文件夹里面是一个简单的接入示例，该示例仅供参考。

当前版本，不需要微信的 SDK，可以正常调用微信支付

## 版本要求
iOS SDK 要求 iOS 7.0 及以上版本

## 接入方法

**安装**

**使用CocoaPods**

1. 在 Podfile 添加 `pod 'XPaySDK'`默认会包含支付宝、微信和点指扫码、点指快捷支付

2. 运行 `pod install`

3. 从现在开始使用 `.xcworkspace` 打开项目，而不是 `.xcodeproj`

4. 添加 URL Schemes：在 Xcode 中，选择你的工程设置项，选中 "TARGETS" 一栏，在 "Info" 标签栏的 "URL Types" 添加 "URL Schemes"，如果使用微信，填入所注册的微信应用程序 id，如果不使用微信，则自定义，允许英文字母和数字，首字母必须是英文字母，建议起名稍复杂一些，尽量避免与其他程序冲突。

5. XPaySDK 打开 Debug 模式，打印出 log，方便调试。关闭方法：`[XPaySDK setDebugMode:NO];`

**手动导入**

1. 获取 SDK

	下载 SDK, 里面包含了 lib 文件夹和 example 文件夹。lib 文件夹里面是 SDK 的文件。

2. 依赖 Frameworks：

	```
	CFNetwork.framework
	SystemConfiguration.framework
	Security.framework
	QuartzCore.framework
	CoreTelephony.framework
	libc++.tbd
	libz.tbd
	libsqlite3.0.tbd
	libstdc++.tbd
	CoreMotion.framework

	```

3. 添加 URL Schemes：在 Xcode 中，选择你的工程设置项，选中 "TARGETS" 一栏，在 "Info" 标签栏的 "URL Types" 添加 "URL Schemes"，如果使用微信，填入所注册的微信应用程序 id，如果不使用微信，则自定义，允许英文字母和数字，首字母必须是英文字母，建议起名稍复杂一些，尽量避免与其他程序冲突。

4. 添加 Other Linker Flags：在 Build Settings 搜索 Other Linker Flags ，添加 -ObjC。

5. XPaySDK 打开 Debug 模式，打印出 log，方便调试。关闭方法：`[XPaySDK setDebugMode:NO];`

**额外配置**

1. iOS 9 以上版本如果需要使用支付宝和微信渠道，需要在 `Info.plist` 添加以下代码：

	```
	<key>LSApplicationQueriesSchemes</key>
	<array>
    <string>weixin</string>
    <string>wechat</string>
    <string>alipay</string>
    <string>alipays</string>
    <string>mqq</string>
	</array>
	```

2. iOS 9 限制了 http 协议的访问，如果 App 需要访问 http://，需要在 Info.plist 添加如下代码：

	```
	<key>NSAppTransportSecurity</key>
	<dict>
    <key>NSAllowsArbitraryLoads</key>
    <true/>
	</dict>
	```
3. 如果编译失败，遇到错误信息为：
	
	```
	XXXXXXX does not contain bitcode. You must rebuild it with bitcode enabled (Xcode setting ENABLE_BITCODE), obtain an updated library from the vendor, or disable bitcode for this target.
	```
	请到 Xcode 项目的 `Build Settings` 标签页搜索 `bitcode`，将 `Enable Bitcode` 设置为 NO。
	
## 注意事项
* 请勿直接使用客户端支付结果作为最终判定订单状态的依据，支付状态以服务端为准!!!在收到客户端同步返回结果时，请向自己的服务端请求来查询订单状态。



