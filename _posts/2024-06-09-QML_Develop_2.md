---
title: 动手制作QML应用程序 2：信号与槽
author: guoao
category: [笔记]
tags: [计算机, 笔记, 桌面开发, C++, QML, Qt6]
image:
   path: https://mir-s3-cdn-cf.behance.net/project_modules/1400_opt_1/2f49cf90337827.5e158de3cb6a8.jpg
---

> 上一篇：[动手制作QML应用程序 1：QML与C++的交互](https://blandalpha.github.io/posts/QML_Develop_1)
{: .prompt-info}

QML作为Qt开发库中的新兴编程语言，其声明式的架构非常适合有 HTML 等设计基础的朋友（比如我）。其更灵活的排版，更强大的组件，更现代化的外观让我产生了兴趣。
 
苦于没有太多中文教程，因此我将基于上方的 QML 教程以及个人开发情况做出总结。

本人在文中将使用 Qt6.7 作为基础，着重于开发好看的 GUI 应用程序，因此会将重点放在如何让你的 *黑框框* 变成 *窗口式程序* 上。

## 什么是信号和槽？

要实现UI的交互，大致可以分为以下三个步骤：

1. 用户做出动作；
2. 界面发送相关信号；
3. 程序接收信号并做出操作。

对于这样一个模式，我们可以使用 **信号 - 槽** (Signal - Slot) 的方式进行解决。

### 信号 (Signal)

在第 2 步中完成的信号发送，我们很容易地能称之为 **信号**。信号是对象内部状态变化的通知，这个通知是全局性的，任何连接到这个信号的对象都可以感受到该信号的发送。

> 一个信号可以绑定多个对象，在信号发送时将会一起触发。
{: .prompt-tip}

### 槽 (Slot)

在第 3 步完成的操作，则被称之为 **槽**。槽是一个可以响应信号的函数，槽函数在连接的信号发射时会被调用。

基础概念解释完了，现在开始正式使用吧！

---

## 开始使用信号与槽！

首先，前端界面元素之间可以使用信号进行通信；其次，QML 和 C++ 之间也可以互相通信。而且 Qt 本身已经自带了许多简单常用的信号和槽函数，如：

```qml
onColorChanged: {
    // Do something
}
```

内部编写的函数将在颜色改变时被触发。

不过这些都是系统发送信号，系统调用槽函数，局限性较大。

## 那如何自己定义自己的信号与槽？

### QML 端的交流

首先在 QML 里定义一个信号：

```qml
Window {
    // ...
    signal testSignal(string s, int value)
}
```

这里我们定义了一个名为 `testSignal` 的信号，并且传递了两个参数： `string` 类型的 `s` 和 `int` 类型的 `value`。

想要发送这个信号，就要在某些情况下触发它。最简单的情况就是点击按钮：

```qml
Button {
    width: 48
    height: 24
    onClicked: {
        testSignal("Canis", 2024)
    }
}
```

当我们按下按钮以后，就会有一个名为 `testSignal` 的信号，带着 `"Canis"` 和 `2024` 两个参数在浩瀚无垠的软件空间里面遨游了。可惜的是，没有一个人愿意接受他。怎么办呢？

有两种方法可以让别的组件接收信号：

#### 第一种方法（不常用）

> 请不要使用第一种方法，实际工程中不会这样用。仅作了解学习即可!
{: .prompt-warning}

首先定义当组件完成创建后绑定信号，并定义触发的函数：

```qml
function func(str, intValue) {
    console.log(str, intValue)
}

Component.onCompleted: {
    testSignal.connect(func)
}
```

这里使用 `connect()` 绑定了 `testSignal` 。当组件接收到 `testSignal`，就会触发函数 `func()` ，并传入这两个值。

此时点击按钮，触发信号，调用函数，最后控制台输出：

```bash
qml: Canis 2024
```

#### 第二种方法（常用）

先看看我们现在有什么：

```qml
Window{

    signal testSignal(string s, int value)

    function func(str, intValue) {
        console.log(str, intValue)
    }

    Button {
        width: 48
        height: 24
        onClicked: {
            testSignal("Canis", 2024)
        }
    }
}
```

一个信号 `testSignal`，一个用于触发的函数 `func` ，还有一个发送信号的按钮 `Button`。只差连接的方法啦！

在Qt中，官方提供了一个名为 `Connections` 的控件帮忙处理信号：

```qml
Connections {
    enabled: bool
    ignoreUnknownSignals: bool
    target: Object
}
```

可以看到 `Connections` 拥有三个参数，这里只讲最后一个 `target`。

`target` 定义了发送信号的 `Object` 是谁。例如：

```qml
Window{
    // ...
    Connections {
        target: window
        function onTestSignal(strValue, intValue) {
            console.log(strValue, intValue)
        }
    }
}
```

后面我们还定义了一个 `onTestSignal` 的函数，它会自动找到 `testSignal` 函数并传递参数。

> 命名要求是 `function + on + 信号名 + 参数名称`
>
> 信号名首字母要记得**大写**，不然会报错！
{: .prompt-tip }

### QML 与 C++ 的交流

#### QML 发送信号绑定到 C++

在开始之前，我们先尝试在 QML 端直接调用一个 C++ 的函数。

##### 如何在 QML 端直接调用一个 C++ 的函数？

假如我们定义了一个类 `MyObject` ，我想在按下按钮的时候直接执行其中的一个函数 `func()` 。如何操作？

目前的 QML 文件如下：

```qml
Window{
    // ...
    Button {
        onClicked: {
            // 这里如何调用 MyObject 中的 func() 函数呢？
        }
    }

    MyObject {
        id: myobj
    }
}
```

假如你想直接使用 `myobj.func()` ，那你会遇到一个大大的报错：

```
TypeError: Property 'func' of MyObject (0x...) is not a function
```

解决方法也很简单，直接在 `myobject.h` 文件中，在 `func` 的声明前添加一句宏 `Q_INVOKABLE`：

```cpp
public:
    // ...
    Q_INVOKABLE void func();
```

加上这个宏以后，当前这个函数就能被 QML 端访问了，但前提是该**对象已经被注册过了**。

##### 信号和槽的绑定

假如在 QML 端有一个信号：

```qml
Window {
    signal qmlSignal(int i, string s)

    Button {
        onClicked: {
            qmlSignal(2024, "CanisAlpha")
        }
    }
}
```

我们希望 C++ 端的 `MyObject` 接收，因此我们可以在 `myobject.h` 中声明一个公共的槽：

```cpp
public:
    // ...

public slots:
    void cppSlots(int i, QString s) {
        qDebug() << __FUNCTION__ << "  " << i << "  " << s;
    }
```

此时 C++ 端就多了一个名为 `cppSlots` 的槽了。他会输出函数名称， `i` 的值， `s` 的值。

那么如何绑定？

第一种方法当然是 `Connections` ：

```qml
Window {
    Connections {
        target: window
        function onQmlSignal(i, s) {
            myobj.cppSlot(i, s)
        }
    }
}
```

另一种方式就是在组件完成创建时使用 `.connect()` 绑定：

```qml
Window{
    Component.onCompleted: {
        qml.Signal.connect(myobj.cppSlot)
    }
}
```

可以发现，两种方法都是一样的，都可以正常输出：

```
MyObject::cppSlot    2024    "CanisAlpha"
```

#### C++ 发送信号到 QML

如果按照上面的说法，假如我要使用 `MyObject` 里面的函数，那我一定要创建一个对象吗？答案是否定的。


在上一讲中，我们使用 `qmlRegisterType()` 注册了对象，这种方法一定要通过创建对象来定义一个 `Object`，类似于模板。而 QML 还为我们提供了另一个方法： `qmlRegesterSingletonInstance()`。这种方法可以创建一个**全局的单例**。例如：

```cpp
qmlRegesterSingletonInstance("MyObj", 1, 0, "MyObject", MyObject::getInstance());
```

> 单例（Singleton）是一种特殊的*对象*，这个对象在你的程序里只有一个。就像你家的大钟，每个房间的人都可以看到它，但整个家里只有这一个钟。
> 而一般的类就像：
> 
> - 你有一种叫做“小汽车”的玩具。
> - 你告诉所有小朋友：“大家可以来拿这种小汽车玩具。”
> - 于是，每个小朋友都拿到了自己的小汽车，可以各自玩自己的小汽车。
>
> `qmlRegisterType`：就像让每个人都可以拿一个小汽车玩具，每个人都有自己的。
> `qmlRegisterSingletonInstance`：就像全家人一起分享一个大机器人，大家都用同一个。
> 
> 这两种方式提供了不同的“玩具”管理方法，适合不同的场景：如果每个人都需要自己的“玩具”，用`qmlRegisterType`；如果大家需要一起分享一个“玩具”，用`qmlRegisterSingletonInstance`。
> 
> *解释来自 ChatGPT-4o ，可能会有误。* 
{: .prompt-info}