# Flutter集成测试

## 一、环境安装

### 1.1、Mac环境安装([文档](https://flutter.dev/docs/get-started/install/macos#downloading-straight-from-github-instead-of-using-an-archive))

#### 1.1.1、安装git

官方教程：https://git-scm.com/download/mac

#### 1.1.2、安装Flutter

##### Flutter环境

1、通过git或者直接在官网下载flutter压缩包，如果是下载压缩包需要解压提取文件。

2、打开.bash_profile或者.bashrc文件，增加全局变量`export PATH="$PATH:[下载的flutter文件目录]/flutter/bin"`。

3、运行`flutter doctor`检查依赖安装情况。

##### 平台构建

###### IOS平台

1、下载Xcode。

2、配置xcode命令行配置

```
sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer
sudo xcodebuild -runFirstLaunch
```

3、运行命令`sudo xcodebuild -license`，确保Xcode许可协议已签署。

4、如果需要部署到真机上面，参考文档：https://flutter.dev/docs/get-started/install/macos#deploy-to-ios-devices。

###### Android平台

1、下载[Android studio](https://developer.android.com/studio)，根据安装向导进行安装。

2、安装Android虚拟机（https://flutter.dev/docs/get-started/install/macos#set-up-the-android-emulator）

3、Android真机运行教程（https://flutter.dev/docs/get-started/install/macos#set-up-your-android-device）

### 1.2、window环境安装（[文档](https://flutter.dev/docs/get-started/install/windows)）

1、通过git方式安装Flutter，在目录下执行`git clone https://github.com/flutter/flutter.git -b stable`。

2、更新全局变量，文档：https://flutter.dev/docs/get-started/install/windows#update-your-path

3、运行命令`flutter doctor`查看平台依赖。

4、安装Android studio流程，https://flutter.dev/docs/get-started/install/windows#android-setup

## 二、依赖

### 2.1、Vscode

1、下载安装Vscode，官方文档：https://code.visualstudio.com/。

2、在Vscode中搜索并安装Flutter，dart插件。

### 2.2、下载骁龙项目

1、使用git获取骁龙项目：git clone ssh://git@gitlab.xiaopeng.local:10022/dragon/xp-app-xdragon.git

### 2.3、Flutter测试依赖

1、打开骁龙项目pubspec.yaml文件，在dev_dependencies下添加flutter_driver和test依赖。

```dart
dev_dependencies:
	flutter_driver:
		sdk:flutter
   test:
```

2、添加好依赖之后，在vscode中执行get packgages。

## 三、测试用例

### 3.1、创建测试文件

1、项目根目录下创建test_driver文件夹。

2、新增app.dart文件，作为flutter drvier命令的入口文件，主要作用是在启动应用程序前，使用flutter_driver插件

```dart
import 'package:xdragon/main.dart';
import 'package:flutter_driver/driver_extension.dart';
void main() {
  enableFlutterDriverExtension();
  app.main();
}
```

3、新增app_test.dart文件，编写测试功能用例。

```dart
import 'package:flutter_driver/flutter_driver.dart';
import 'package:test/test.dart';
void main() {
  group('功能描述', () {
    FlutterDriver driver; // 负责跟应用交互实例
    setUpAll(() async{
      // 在用例运行前连接上应用;
      driver = await FlutterDriver.connect();
    });
    tearDownAll(() {
			if (driver != null) {
        // 用例执行完成之后，需要关闭通道。
        driver.close();
      }
    })
    test('描述xxx', () async {
      // 获取想要操作的widget
    	final textFinder = find.byType("TextField");
      await driver.tap(textFinder); // 实现点击事件
      expect(await driver.getText(textFinder), 'xxx'); // 对比当前获取值跟期望值。
    });
  })
}
```



### 3.2、编写测试常用语法

1、获取元素方式

```dart
// 通过Text或者EditableText Widget组件获取
final textFinder = find.text('姓名');
// 通过唯一key值获取
final keyFinder = find.byValueKey('name');
// 通过runtimeType获取
final typeFinder = find.byType('TextField');
```

2、交互事件

```dart
// 点击事件
final buttonFinder = find.byValueKey('立即预约');
driver.tap(buttonFinder);
// 滚动事件
driver.scroll(finder, dx, dy, duration); //finder被滚动的元素、dx横向滚动距离、dy纵向滚动距离、duration滚动花费时间。
driver.scrollIntoView(finder);// 滚动finder所在的祖先可滚动元素，直到finder出现在可视范围。
driver.scrollUntilVisible(scrollable, item);// 滚动scrollable元素，直到item出现在可视范围，并且可配置想要item出现的位置，参数参考代码提示。
```



### 3.3、运行测试用例

测试用例编写完成之后，通过在终端运行下面代码启动应用

```dart
flutter driver --target=test_driver/app.dart
```



## 四、需要掌握的技术栈

### 4.1、test第三方库

资源文档：https://pub.dev/packages/test

### 4.2、flutter_driver第三方库

资源文档：https://api.flutter.dev/flutter/flutter_driver/flutter_driver-library.html

### 4.3、matcher库（进阶）

资源文档：https://pub.dev/documentation/matcher/latest/matcher/matcher-library.html

### 4.4、项目相关语法

dart：https://dart.dev/samples

## 五、问题归纳

### 5.1、本地图片不能加载

描述：在使用flutter_driver命令执行集成测试的时候，启动项目之后，本地的图片都不能加载出来。

原因解析：在读取静态资源的时候，如果超过10kb，资源加载会在另外的isolate上面进行，使的test在资源加载之前执行完成，导致最终获得空资源文件。

解决方案：在main.dart文件中使用DefaultAssetBundle绑定资源。

### 5.2、不支持键盘事件

描述：在搜索活动的时候，输入搜索内容之后，无法触发键盘的搜索按钮。

原因解析：flutter_driver官方目前不直接支持键盘事件。

解决方案：通过FlutterDriver requestData方法来跟app.dart通信，实例TestTextInput对象。
![image-20210105112309371](/Users/tanlianghao/Library/Application Support/typora-user-images/image-20210105112309371.png)

## 六、不支持的功能

### 6.1、不支持选择/上传图片