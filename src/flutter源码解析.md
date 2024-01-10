Flutter疑问

1、flutter怎么启动应用。
2、widget、Element、renderobject之间的关系，且之间的转化关系。
3、为什么lib/main.dart 需要使用main()函数来包括。

Flutter源码

通过在main.dart文件中，使用runApp函数，该函数会调用到WidgetsFlutterBinding类的scheduleAttactRootWidget方法

```dart
void main() {
  runApp(Xdragon())
}
void runApp() {
  // 确保获取到实例
  WidgetFlutterBinding.ensureInitalized()
    ..scheduleAttachRootWidget() // WidgetBinding类方法
    ..scheduleWarmUpFrame()
}
void scheduleAttachRootWidget() {
  // 设置的倒数时间为0，则使用Timer.run方法简写。
  Timer.run((){
    // 把widget附加到renderViewElement上。
		attachRootWidget(Widget app);
  })
}
/// BuildOwner类 用来管理widget框架，比如widget需要更新、热重载等。
///
void attachRootWidget(Widget rootWidget) {
  _readyToProduceFrames = true; // 变量用来做状态判断。
	_renderViewElement = RenderObjectToWidgetAdapter<RenderBox>(
    container: renderView,
    debugShortDescription: '[root]',
    child: rootWidget,
  ).attachToRenderTree(buildOwner, renderViewElement as RenderObjectToWidgetElement<RenderBox>);
}
/// 刚启动应用的时候，element=null
RenderObjectToWidgetElement<T> attachToRenderTree(BuildOwner owner, [ RenderObjectToWidgetElement<T> element ]) {
    if (element == null) {
      owner.lockState(() {
        element = createElement(); // 创建一个element实例。
        assert(element != null);
        element.assignOwner(owner);// 设置owner
      });
      // buildScope确认需要更新Scope，调用给出的回调。然后使用scheduleBuildFor更新被标记为dirty的element
      owner.buildScope(element, () {
        element.mount(null, null);
      });
      SchedulerBinding.instance.ensureVisualUpdate();
    } else {
      element._newWidget = this;
      /// markNeedBuild用来标记element为dirty，并把dirty的element添加到global中widget列表中，等待下一帧rebuild。
      element.markNeedsBuild();
    }
    return element;
  }
// element.mount方法调用的是RenderObjectToWidgetElement中的mount方法。
  @override
  void mount(Element parent, dynamic newSlot) {
    assert(parent == null);
    super.mount(parent, newSlot); // 调用父类RenderObjectElement.mount方法
    _rebuild();
  }
// RenderObjectElement
@override
  void mount(Element parent, dynamic newSlot) {
    super.mount(parent, newSlot); // 父类中的mount作用是更新_inheritedWidgets
   ...
    _renderObject = widget.createRenderObject(this); // widget的类型是RenderObjectToWidgetAdapter
   ...
     // 找寻祖先RenderObjectElement，并把当前的RenderObject插入进去。
    attachRenderObject(newSlot);
    _dirty = false;
  }

void _rebuild() {
    try {
      _child = updateChild(_child, widget.child, _rootChildSlot);
    } catch (exception, stack) {
      ...
    }
  }
// Element中的updateChild方法
Element updateChild(Element child, Widget newWidget, dynamic newSlot) {
	...
    newChild = inflateWidget(newWidget, newSlot); // widget转化成element
  ...
}
// Element的inflateWidget方法
Element inflateWidget(Widget newWidget, dynamic newSlot) {
    ...
    final Element newChild = newWidget.createElement(); // newWidget为传入的Widget类。
  /// 这个真是调用到代码中的StatelessElement.createElement方法。生成一个element
    ...
    newChild.mount(this, newSlot);// newchild为element子类
  /// 调用mount 其实调用到的是ComponentElement类中的mount方法。
    return newChild;
  }
// ComponentElement
void mount(Element parent, dynamic newSlot) {
    super.mount(parent, newSlot);
    _firstBuild();
  }
  void _firstBuild() {
    rebuild(); // rebuild调用的是父类ELement中的方法。
  }
// Element
void rebuild() {
    ...
    performRebuild(); // 在ComponentElement子类中实现的方法。
    ...
  }
void performRebuild() {
    ...
    Widget built;
    try {
      ...
      built = build(); // 实际调用的是子类中的build方法，比如StatelessWidget中的build
      ...
    } catch (e, stack) {
      ...
    } finally {
      // We delay marking the element as clean until after calling build() so
      // that attempts to markNeedsBuild() during build() will be ignored.
      _dirty = false;
    }
    try {
      _child = updateChild(_child, built, slot);
    } catch (e, stack) {
      ...
      _child = updateChild(null, built, slot);
    }
    if (!kReleaseMode && debugProfileBuildsEnabled)
      Timeline.finishSync();
  }
```

