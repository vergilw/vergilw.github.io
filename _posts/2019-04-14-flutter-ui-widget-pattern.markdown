---
layout: post
title: Flutter UI开发Widget选型
date: 2019-04-14
---

> 刚接触Flutter的时候，UI开发常常会遇到选用StatefulWidget还是StatelessWidget的问题。良好的设计对于程序性能有一定帮助，所以Flutter平台下的开发者应该掌握这个知识点。

## State介绍
在选择StateFul和Stateless之前，先要了解清楚State。

一个StatefulWidget类会对应一个State类，State表示与其对应的StatefulWidget要维护的状态，State中的保存的状态信息可以：

1. 在widget build时可以被同步读取。
2. 在widget生命周期中可以被改变，当State被改变时，可以手动调用其setState()方法通知Flutter framework状态发生改变，Flutter framework在收到消息后，会重新调用其build方法重新构建widget树，从而达到更新UI的目的。

State中有两个常用属性：

1. widget，它表示与该State实例关联的widget实例，由Flutter framework动态设置。注意，这种关联并非永久的，因为在应用声明周期中，UI树上的某一个节点的widget实例在重新构建时可能会变化，**但State实例只会在第一次插入到树中时被创建，当在重新构建时，如果widget被修改了，Flutter framework会动态设置State.widget为新的widget实例**。
2. context，它是BuildContext类的一个实例，表示构建widget的上下文，它是操作widget在树中位置的一个句柄，它包含了一些查找、遍历当前Widget树的一些方法。每一个widget都有一个自己的context对象。

所以State对象Stateful对应的State对象只会创建一次，哪怕渲染树发生了改变，修改了当前Stateful的Widget，State还是不会变化，只是将Widget指向了新的Widget。

## Stateful与Stateless的选择
StatefulWidget是解决问题的超集，它同样能解决StatelessWidget可以解决的问题，因为StatefulWidget可以拥有一个空的State对象。但是，实现StatefulWidgets的语法比StatelessWidget更简单。Flutter框架中设计了StatelessWidget，就是可以使用不那么详细的机制解决更简单的问题因此（那些不需要状态的问题）。
所以说选择了合适的Widget，提升开发效率是显而易见的。

### 三种常用情况下的选型
1. someDate由于父Widget发生的事情而发生变化。例如，FoobarWidget的功能只是显示日期，那么当父Widget需要更新日期时，它可以创建具有新状态的FoobarWidget的新实例。在这种情况下，可以只使用一个简单的StatelessWidget。
```dart
class FoobarWidgetA extends StatelessWidget {
  final DateTime someDate;

  FoobarWidget({ Key key, this.someDate }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return new Text("the date is ${someDate}");
  }
}
```
2. someDate更改是因为自身FoobarWidget发生的事情，父Widget需要知道更改后的日期。例如，如果FoobarWidget的功能是让用户选择日期，则需要通知父Widget用户选择的日期。在这种情况下，父Widget提供someDate值和FoobarWidget在想要更改值时应调用的callback。父Widget接收新值，自身调用setState，并使用新值重建FoobarWidget，更新UI。
```dart
class FoobarWidgetB extends StatelessWidget {
  final DateTime someDate;
  final ValueChanged<DateTime> onChanged;

  FoobarWidget({ Key key, this.someDate }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return new Column(
      children: [
        new Text("the date is ${someDate}"),
        new RaisedButton(
          onPressed: () {
            onChanged(someDate.add(new Duration(day: 1)));
          },
          child: new Text("NEXT"),
        ),
      ],
    );
  }
}
```
3. someDate发生了变化，因为自身FoobarWidget发生了一些事情，但父Widget不需要知道更改后的日期。例如，如果FoobarWidget的功能是展示不同日期探索月球的各个阶段，用户可以选择不同日期来查看。在这种情况下，比较好的模式是“initial value pattern”。在此模式中，父Widget提供初始值，当State对象初始化时，FoobarWidget将复制到其State对象中的可变字段someDate中。当用户与FoobarWidget交互时，State对象中的字段发生改变，setState让FoobarWidget使用新值重建。永远不会通知父Widget修改后的新值，如果父Widget使用新的初始值重建，则FoobarWidget会忽略新的初始值，因为它的State初始化过了。
```dart
class FoobarWidgetC extends StatefulWidget {
  final DateTime initialDate;

  FoobarWidget({ Key key, this.initialDate }) : super(key: key);

  @override
  _FoobarWidgetState createState() => new _FoobarWidgetState();
}

class _FoobarWidgetCState extends State<FoobarWidgetC> {
  DateTime someDate;

  @override
  void initState() {
    super.initState();
    someDate = widget.initialDate;
  }

  void _advanceDate() {
    setState(() {
      someDate = someDate.add(new Duration(day: 1));
    });
  }

  @override
  Widget build(BuildContext context) {
    return new Column(
      children: [
        new Text("the date is ${someDate}"),
        new RaisedButton(
          onPressed: _advanceDate,
          child: new Text("NEXT"),
        ),
      ],
    );
  }
}
```
---

#### 更多资料
1. [官方文档StatefulWidget-class](https://docs.flutter.io/flutter/widgets/StatefulWidget-class.html)
2. [官方文档StatelessWidget-class](https://docs.flutter.io/flutter/widgets/StatelessWidget-class.html)
3. [flutter-dev forum](https://groups.google.com/forum/#!msg/flutter-dev/zRnFQU3iZZs/ThAfrMfyBwAJ)
4. [Effective Flutter](https://github.com/flutter/flutter/issues/7044)
