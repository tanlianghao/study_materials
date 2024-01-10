## Flutter 动画

### 一、动画的基本概念

#### 1.1 相关概念

##### 1.1.1 帧

帧是作为动画中最小单位的画面，一帧就是一个静止的画面。帧大致可分为两类，关键帧和过渡帧。关键帧可以看作是动画的起始状态，而过渡帧则是自动完成并插入到关键帧之间的部分。比如Flutter中的补间动画，一般都是设置一个起始状态就可以。如下面的代码，设置了宽高，通过setstate再来变化从而尝试动画效果。

```dart
AnimatedContainer(
              duration: Duration(seconds: 1),
              width: size,
              height: size,
              curve: Curves.easeIn,
              padding: const EdgeInsets.all(20.0),
              decoration: BoxDecoration(
                  color: colors, borderRadius: BorderRadius.circular(radius)),
              child: FlutterLogo(),
            )
```

##### 1.1.2 帧数与FPS

帧数，帧的数量。FPS，即每秒显示的帧数。一般到60FPS。对于Flutter来说目标是60FPS，或者120FPS。

##### 1.1.3 插值器

插值器主要的作用是实现动画的过渡效果，其实就是通过一个数学函数来实现设置的属性值从初始值到结束值的变化规律。各方平台都设置了一些选项给到开发者快速接入，比如Flutter中可以选择ease等过渡效果。

### 二、Flutter动画的分类

#### 2.1 补间动画

就是上面说到的给定起始值作为关键帧，给定插值器，系统自动补充上过渡帧实现的动画。

#### 2.2 基于物理的动画

一种遵循物理学定律的动画形式。比如球从高度落下，会受到速度跟加速度的影响。

### 三、Flutter动画实例

#### 3.1、隐式动画

文献：https://medium.com/flutter/flutter-animation-basics-with-implicit-animations-95db481c5916

#### 3.2、自定义隐式动画

1、当找不到符合条件的内置隐式动画时，可以使用TweenAnimationBuilder来自定义动画。
2、并不需求一定写在statefulWidget组件内。
3、可以动态改变Tween的值。
4、Tween的变化总是从当前值->最终值来实现动画。
5、能支持动画完成回调。
6、支持child属性，已优化性能。

#### 3.3、显示动画

##### 3.3.1、区别

显示动画跟隐式动画的区别在于，前者能重复的动画，而隐式动画给定起始值，从开始到结束状态的变化

##### 3.3.2、内置显示动画

内置的显示动画很多，通常是xxxTransition的命名形式。需要使用到AnimationController控制器，通过它来管理动画。
FadeTransition、ScaleTransition、SizeTransition

##### 3.3.3、自定义显示动画

自定义方式有两种：AnimatedBuilder和AnimatedWidget，builder能更好的优化性能。

### 四、Flutter动画库

#### 4.1 Animation库

Animation.dart定义了Flutter动画的四种状态，以及核心的抽象类Animation

```dart
enum AnimationStatus {
  /// 动画的开始状态
  dismissed,
  /// 从头到尾播放
  forward,
  /// 从尾到头播放
  reverse,
  /// 动画的结束状态
  completed,
}
// 以及该抽象类定义了一些通用的方法
/// 动画值的回调
void addListener(VoidCallback listener);
void removeListener(VoidCallback listener);
/// 状态回调
void addStatusListener(AnimationStatusListener listener);
void removeStatusListener(AnimationStatusListener listener);
// 用来驱动动画的函数
Animation<U> drive<U>(Animatable<U> child) {
    assert(this is Animation<double>);
    return child.animate(this as Animation<double>);
  }
```

#### 4.2 Curve类

Curve类作为插值器，作用是映射时间与数值之间关系的接口，从初始值到结束值变化的规律。curve.dart定义了常用的插值器，如easeIn

```dart
// Curve继承自ParametricCurve类
abstract class ParametricCurve<T> {
  const ParametricCurve();

  T transform(double t) {
    assert(t != null);
    assert(t >= 0.0 && t <= 1.0, 'parametric value $t is outside of [0, 1] range.');
    return transformInternal(t);
  }

  @protected
  T transformInternal(double t) {
    throw UnimplementedError();
  }

  @override
  String toString() => objectRuntimeType(this, 'ParametricCurve');
}
// curve.dart中定义的插值器，官方推荐是重写transformInternal方法
class _Linear extends Curve { //线性插值器
  const _Linear._();

  @override
  double transformInternal(double t) => t;
}
```

#### 4.3 估值器（tween）

根据不同的输入，产出不同的数值，并通过重载transform来实现不同的估值器，从初始值到结束数值的变化

```dart
class Tween<T extends dynamic> extends Animatable<T> {
  Tween({
    this.begin,
    this.end,
  });
  T? begin;
  T? end;
  @protected
  T lerp(double t) {
    assert(begin != null);
    assert(end != null);
    return begin + (end - begin) * t as T;
  }
  @override
  T transform(double t) {
    if (t == 0.0)
      return begin as T;
    if (t == 1.0)
      return end as T;
    return lerp(t);
  }
  @override
  String toString() => '${objectRuntimeType(this, 'Animatable')}($begin \u2192 $end)';
}

class ColorTween extends Tween<Color?> {
  ColorTween({ Color? begin, Color? end }) : super(begin: begin, end: end);
  @override
  Color? lerp(double t) => Color.lerp(begin, end, t);
}
```

#### 4.4 控制器 animation_controller

控制动画的播放或者停止，设置动画的值、创建基于物理的动画效果。
默认animationController会产生线性的值。

##### 4.4.1 生命周期

```dart
// State.initState中创建animationController，在State.disponse中注销，为了防止资源泄露。
@override
ininState() {
  AnimationController _animation = AnimationController(vsync: this, duration: Duration(second: 1))
}
// 在初始化animationControlloer的时候，需要用到TickerProvider
```

##### 4.4.2 TickerProvider

主要作用是获取每一帧刷新的通知。常用的两个类：TickerProviderStateMixin 和 SingleTickerProviderStateMixin，当只需要一个Ticker时使用SingleTickerProviderStateMixin效率更高些。

### 五、动画类型选择

![image-20201128183458540](/Users/tanlianghao/Library/Application Support/typora-user-images/image-20201128183458540.png)