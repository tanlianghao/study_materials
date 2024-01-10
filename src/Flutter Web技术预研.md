## Flutter Web技术预研

### 创建新的flutter web项目

1、通过`flutter channel beta`命令切换flutter插件到beta版本。
2、通过`flutter config --enable-true`来启用web。启动一次，修改的是～/.flutter_settings文件。
3、使用vscode获取flutter命令来创建flutter web项目。
4、使用flutter build web来打包。

#### 启动flutter web应用

1、同app启动方式类似， 使用IDE启动或者命令行flutter run -d chrome。

#### 部署构建Flutter web应用

1、flutter build web，指定生成web代码。使用dart2js打包生产环境内容。

#### 向现有的应用增加web支持

1、使用flutter create . 来向现有的项目中增加web支持。

#### 区别

flutter web不能使用Dart:io

### flutter web优缺点

#### 优点

1、因为是dart语法编写，后续切换到app方便。
2、flutter web技术使得现有的应用能打包成PWA应用，为现有的程序提供web功能。

#### 缺点

1、打包之后体积大。应用白屏时间很长。
2、打包之后的代码，在chromedebug不方便找到错误源。
3、目前只支持beta版本，要在正式环境使用，需要多机型覆盖测试。

### Web FAQ

1、目前flutter 只在beta版本上支持Flutter web技术。所以在部署到生产环境的时候需要在各平台测试通过。
2、什么情况下适合使用flutter web。
连接pwa；动态将内容嵌入已有的网站。
3、支持flutter的技术的web浏览器。包括chrome、safari、edge、firefox。
4、推荐使用最新版本的flutter插件、dart插件。
5、不能使用dart:io插件，因为浏览器不支持查看文件系统，如果需要使用到网络请求，请使用http包，因为在浏览器中，是浏览器来控制请求头。

### [PWA应用](/Users/tanlianghao/Documents/PWA资料.md)



