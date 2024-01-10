### 安装flutter遇到的问题

1、在安装flutter的使用需要指定环境变量path。

```
export PATH="$PATH:`pwd`/flutter/bin"
```

2、安装xcode之后，运行flutter doctor会出现xcode插件cocoapods未安装情况，使用

```
sudo gem install cocoapods
```

3、使用vscode运行flutter项目，需要先安装flutter插件。如果出现未找到flutter SDK情况，可以手动运行flutter bin文件。
4、根据flutter教程，使用export PATH=$PATH: ${pwd}/flutter/bin来设置flutter的环境变量，会出现重新打开终端，flutter 命令not found情况，需要更新～/.zshrc文件，使用source ~/.bash_profile，在.bash_profile文件中增加配置项

```
export PATH=${PATH}:/Users/tanlianghao/works/flutter/bin:$PATH
export PUB_HOSTED_URL=https://pub.flutter-io.cn
export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn
```

5、在使用flutter doctor的时候，出现android studio 证书安装有问题，该IDE上Flutter、Dart插件没有安装。使用flutter doctor --android-licenses 的时候出现如下错误

![image-20200921164807003](/Users/tanlianghao/Library/Application Support/typora-user-images/image-20200921164807003.png)

原因很明显，是android sdkmanager tool没有。打开android staudio，如下，安装好sdk tools。最后执行安装证书命令，安装成功。
![image-20200921165127430](/Users/tanlianghao/Library/Application Support/typora-user-images/image-20200921165127430.png)

安装成功之后，flutter doctor检查没有其他问题，启动flutter 程序，选择android虚拟机，启动不成功，报错。
`Exception in thread "main" java.util.zip.ZipException: error in opening zip file`，跟进issues的解决办法[#51195](https://github.com/flutter/flutter/issues/51195#issuecomment-589708062)。重新启动flutter app的时候 如果出现了`Error connecting to the service protocol: failed to connect to`重新启动虚拟机跟flutter IDE。



### Dart语法

1、Null-aware operators

```dart
// ??= 当运算符左侧的值为null时，才会把运算符右侧的值赋值给左侧。
int a;
a ?? = 3;
print(a) // 3;
a ?? = 5;
print(a) // 3
// ?? 运算符，当运算符左侧的不为null，获取到的是左侧的值，否则获取的是右侧的值，相当于||运算符。
print(1??3) // 1
print(null ?? 3) // 3
```

2、条件判断运算符

```dart
// ?. 判断运算符左侧的对象不为空
myObject?.someProperty
// 等价于
(myObject != null) ? myObject.someProperty : null;
// 在写dart语法的时候，使用if判断，判断的条件必须是boolean，不能是字符串之类的。
```

3、对象属性

```dart
// dart语法中对象包括List、set、map 三种类型。
final aListObject = ['a', 'b', 'c'];
final aSetObject = {1,23,5};
final aMapObject = {"myKey": 12};
final aListEmptyObj = <String>[];
final aSetEmptyObj = <int>{};
final aMapEmptyObj = <double, int>{};
```

4、cascades（链式调用）

```dart
class BigObject {
	int aInt = 0;
  String aString = '';
  List<double> aList = [];
  bool _done = false;
  
  void allDone() {
		_done = true;
  }
}

BigObject fillBigObject(BigObject obj) {
  return obj..aInt=1..aString='String!'..aList=[3.0]..allDone();
} 
```

### flutter程序

1、通过vscode可以创建一个新的flutter应用。创建好的项目，入口是在lib/main.dart文件。

```dart
import 'package:flutter/material.dart'; // 导入第三方库
void main() { runApp(MyAPP())} // 应用入口程序。

class MyApp extends StatelessWidget { // StatelessWidget无状态的widget
  @override // 复写方法
  Widget build(BuildContext context) { // 应用启动，都会触发build函数。
    return new MaterialApp({home: MyHomePage}) // 用来设置app首页的相关信息, home用来设置首页页面
  }
}
class MyHomePage extends StatefulWidget { // 继承有状态的Widget
  // stateful widget组件，至少包含两个类，StatefulWidget类，State类持有的状态在widget生命周期中可能会出现变化。
  _MyHomePageState createState() => _MyHomePageState();
}
class _MyHomePageState extends State<MyHomePage> {
  void _incrementCounter() {
    // setState会通知Flutter框架，有状态的变化，会再调用到build函数。
    setState(() {
       _counter++;
    });
  }
  @override
  Widget build();
}
```

2、使用表单提交的时候onsubmit，从子组件穿过来的String格式，最后变成个了List格式？？？
state.didchange()返回调用的是Formelement中的onSaved方法，

```dart
Widget _buildMultisetUploadButton(FormElement e) {
    return MultisetUploadButton(
      placeholder: e.image,
      onSaved: (String value) {
        List _value;
        if (value != null) {
          _value = value.split(',');
        }
        _data[e.name] = _value;
      },
    );
  }
```

3、使用GestureDetector组件的时候，默认的onTap点击区域很小，默认是只在点击对应的Text时才生效。通过设置behavior属性来把点击事件穿透。到整个内容。
4、使用TextField组件的时候，通过conentPadding来设置高度， 默认高度是48，通过decoration: InputDecoration的isDense = true来是高度生效。
5、flutter元素的层级，是跟进子widget的放置位置来决定的。遮罩层的写法，

```dart
Stack(
  fit: StackFit.expand // 没设置的话，遮遮层同样适配机型。
	Container(decoration: BoxDecoration(color: Color(0xFF000000).withOpacity(0.7))),
  ...widget.child,
  Positioned()
)
```

6、Widget组件的传参方式。
7、dart中abstract类 跟mixins的区别。
8、flutter开发中遇到的key作用及使用规范。

```dart
key 是每个widget都能继承的，
class Mykey extends StatefulWidget{
  Mykey({Key key}):super(key: key)
}
Widget的更新机制
// Widget源码
abstract class Widget extends DiagnosticableTree{
  const Widget({this.key});
  final Key key;
  ...
  // Widget不能修改，Element能修改。
	// 用来确定Element是否需要更新，canUpdate返回true，则Element不需要更新，直接替换Widget
  // 为false的是，则Element需要更新，
  static bool canUpdate(Widget oldWidget, Widget newWidget) {
    return oldWidget.runtimeType == newWidget.runtimeType &&
      oldWidget.key == newWidget.key;
  }
}
```

7、在使用Row构建容器的时候，子元素使用Text来设置，内容过长的时候导致容器溢出，原因是Text没规定长度，导致text根据内容来撑开内容宽度，解决版本是设置内容宽度，可以通过Expanded来设置子元素占用宽度比。
8、Flexible的使用情况，当子元素需要最大占父元素的宽度时，较小的时候为自己的宽度，可以通过设置Flexible的fit：FlexFit.loose

### 开发问题反馈

1、vscode为啥选中了Uncaught Exceptions之后，会阻止后续代码的执行。
2、flutter app在真机上测试的方案实现。1.需要app开发者账号，并且在https://developer.apple.com/account/resources/devices/list，增加自己的iphone机UDID号，2.通过Xcode打开flutter项目/ios/Runner.xcworkspace文件，点击Xcode左侧的Runer，在sign栏中选中（如下图），执行第三步新增到团队。3、通过数据线连接，进行调试。

![image-20200908172813338](/Users/tanlianghao/Library/Application Support/typora-user-images/image-20200908172813338.png)

3、ValueNotifier数据传递\状态管理？？？
4、关于组件更新不成功问题。例子1：在做线索状态选择栏的时候，通过setState更加_formdata数据之后，在自组件中，xx方法没有执行。因为组件只在initstate的周期里面执行了函数。最后是通过statefulWidget组件的生命周期didUpdateWidget方法中通过对比新旧widget配置数据，来确认是否执行该方法。

```dart
  @override
  void initState() {
    super.initState();
    if (widget.initialValue != null) _setValue(widget.initialValue);
  }
  @override
  void didUpdateWidget(oldWidget) {
    var option = oldWidget.options;
    if (option != widget.options) {
      _setValue(widget.controller.value);
    }
    super.didUpdateWidget(oldWidget);
  }
  _setValue(String value) {
    for (int i = 0; i < widget.options.length; i++) {
      final option = widget.options[i];
      if (option.value == value) {
        setState(() {
          _index = i;
          _label = option.label;
          _controller = FixedExtentScrollController(initialItem: i);
        });
        break;
      }
    }
  }
```

例子2：线索录入页面，刚进入该页面的时候，线索状态为跟进中状态，选择另外的状态时候，发现线索失败原因更新值不成功。发现是因为FieldInputController的参数Globalkey没拿到currentState的值，原因是该组件这个时候还没挂载成功。通过使用下一帧的功能完善

```dart
// WidgetBinding.instance.addPostFrameCallback方法
   onChanged() {
    if (fieldKey.currentState != null) {
      fieldKey.currentState.didChange(value);
    } else {
      WidgetsBinding.instance.addPostFrameCallback((_) {
        fieldKey.currentState.didChange(value);
      });
    }
  }
```

8、flutter 图片内存占用问题https://juejin.im/post/6844903689254076424#comment

9、Flutter inspector 工具使用，Detail Tree 用来查看各widget之间的关系，containts从父到子传递，layout explorer用来查看widget的样式布局。

10、使用tabBarView来做切换的tab页面时候，使用PagestoreKey来保存页面的滚动信息，保证在切换tab的时候，能记住当前的滚动位置

11、返回数据json类化；使用枚举类型；文件名命名规则+变量命名规则；

### Flutter开发工具

1、性能分析工具
https://dart-lang.github.io/observatory/
2、dav tool

### 面评分析点

1、对业务的理解，做过的项目，效果怎么样。
2、对团队的帮助，基础建设，开发流程优化。

### 骁龙flutter基础组件

1、自定义单选项
2、筛选栏基础widget
3、搜索中的历史，抽离成抽象类。
4、flutter目前不知道android模拟机跑项目，待排查。
5、xdragon现在升级Flutter SDK升级之后，项目跑不起来。

### 骁龙待优化项

1、APP内存占用优化。
2、Flutter 数据共享（InheritedWidget）接入调研。
3、工具页面在sf账号切换的时候需要自动更新线索数据。
4、动画
5、flutter项目升级，flutter SDK升级内容，升级过程可能遇到的坑。
6、flutter to web （重点关注）
7、APP打包大小优化

### Flutter 部署应用流程

#### android包

1、修改应用的icon图片。
2、对app进行签名。
3、使用R8对应用代码进行压缩。默认打release包的时候是使用R8压缩的，如果需要禁用 使用命令参数--no-shrink
4、检查清单文件，路径是/android/app/src/main/AndroidManifest.xml。
5、检查打包文件/android/app/build.gradle
6、打包release包有两种方式，一个是appbundle，一个是apk

### Flutter 升级

1、从Flutter Engine下载Dart SDK

![image-20200929155636756](/Users/tanlianghao/Library/Application Support/typora-user-images/image-20200929155636756.png)

2、构建Flutter tool

![image-20200929155705663](/Users/tanlianghao/Library/Application Support/typora-user-images/image-20200929155705663.png)

![image-20200929155733561](/Users/tanlianghao/Library/Application Support/typora-user-images/image-20200929155733561.png)



![image-20200929155752203](/Users/tanlianghao/Library/Application Support/typora-user-images/image-20200929155752203.png)

#### 升级后的问题

1、通过flutter upgrade升级之后，运行flutter run项目，出现错误`warning: 'sqlite3_wal_checkpoint_v2' is only available on iOS 5.0 or newer `,出现这个问题的解决版本可以先在跟目录运行flutter clean来清除缓存，并在ios目录下执行pod install，
2、运行pod install的时候，出现如下错误
![image-20200929162753501](/Users/tanlianghao/Library/Application Support/typora-user-images/image-20200929162753501.png)
查看之后发现是pubspec.yml文件中path依赖有问题。按提示运行flutter pub get ，会出现path包版本错误情况。修改path包版本为1.7.0
![image-20200929163002184](/Users/tanlianghao/Library/Application Support/typora-user-images/image-20200929163002184.png)

3、需要指定`SDWwebImage:5.0.6`能解决该问题。

![image-20200930144336132](/Users/tanlianghao/Library/Application Support/typora-user-images/image-20200930144336132.png)

4、NIMKit/Lite包的问题，使用这个版本的包需要指定SDWebImage：5.0.6版本才行`pod 'SDWebImage', '~> 5.0.6'`。

![image-20200930144631218](/Users/tanlianghao/Library/Application Support/typora-user-images/image-20200930144631218.png)

5、flutter_cupertino_date_picker依赖的问题，升级flutter之后，date_picker_theme.dart文件中的DiagnosticableMixin类需要改成Diagnosticable。[issue#155](https://github.com/Realank/flutter_datetime_picker/issues/155#)

![image-20200930140521330](/Users/tanlianghao/Library/Application Support/typora-user-images/image-20200930140521330.png)

