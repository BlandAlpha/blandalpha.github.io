---
title: 动手制作QML应用程序 1：QML与C++的交互
author: guoao
category: [笔记]
tags: [计算机, 笔记, 桌面开发, C++, QML, Qt6]
image:
   path: https://mir-s3-cdn-cf.behance.net/project_modules/1400_opt_1/094d8e101478433.5f1fe74fd657d.jpg
---

参考教程： [QML教程 - 落雨薄青衫](https://www.bilibili.com/video/BV1dS4y1u7vN)

> 本教程**不**面向完全0基础的朋友。请先了解最基本的 C++ 以及 QML 的编写再尝试阅读本文。看看上方教程的前 20 讲可能会对你的 QML 有帮助。
{: .prompt-info}

QML作为Qt开发库中的新兴编程语言，其声明式的架构非常适合有 `HTML` 等设计基础的朋友（比如我）。其更灵活的排版，更强大的组件，更现代化的外观让我产生了兴趣。

苦于没有太多中文教程，因此我将基于上方的 QML 教程以及个人开发情况做出总结。

本人在文中将使用 Qt6.7 作为基础，着重于开发好看的 GUI 应用程序，因此会将重点放在如何让你的 *黑框框* 变成 *窗口式程序* 上。

## 使用 QML 与 C++ 开发应用是怎么一回事？

> 相关阅读：[全面认识 Qt Widgets、QML、Qt Quick](https://cloud.tencent.com/developer/article/1843462)
{: .prompt-info }

首先你需要了解下面这三个东西分别是什么。

- **Qt** 作为一个**应用程序开发框架**，它为你提供了几乎所有你需要制作一个软件的工具，就像一个工具箱。

- **QML** 是基于JS的声明式语言（类似HTML），主要侧重于**UI的编写与设计**。其标准库是 **Qt Quick**。

- **C++** 是你编写**主要逻辑**的语言。这里你可以将你的黑框框应用程序照搬过来。

### 那 Qt Widgets 又是什么情况？

之前做 Qt 开发的朋友基本都是使用的 Qt Widgets 作为界面的库。如今 QML/Qt Quick 将起到**代替**的作用，让 Qt 开发迈向更加现代化的未来：**前后端分离**（类似于HTML5 + JS）UI界面只需要搞好显示的功能，而后端只需要负责将数据传给UI正确显示即可。

> 其实这就是后面会提到的 `MVC` 模型。
>
{: .prompt-tip }

## 我能做一个什么软件？

最基础的当然是和数据库的联动啦！学生信息管理、超市商品管理等等……不过就我个人而言，目前我正在尝试完成一个*“布鲁伊动画指南” (Bluey Gallery)*。我构建了一个数据库，里面包含了：

- **角色信息**：角色的中英文姓名、种族、简介、图片链接；
- **剧集信息**：剧集的季、集、简介、图片链接；
- **关系信息**：某一集出现了哪些角色，用于 APP 的查询与生成。

因此我将基于这个数据库进行 APP 的构建。

---

## 来认识常见的软件框架吧！

前置知识大概了解好了，下面正式开始介绍 QML 开发的内容！

一般的 QML 软件会使用**模型 - 视图 - 控制器** (`MVC`) 设计结构进行构建：

```
Model - View - Controller
```

这种设计方式的好处上方已经提到过，就是前后端分离，不再赘述。下面则对这种结构作一点简单的解释：

### 视图 (View)

定义你的软件长什么样子，怎么排版，显示哪些内容。是软件与用户交互的主要场地。

### 模型 (Model)

- 获取 QML 端需要的数据；
- 设置 QML 端的数据；
- 获取 QML 端传回的数据。

### 控制器 (Comtroller)

用于对接外部的控件或者模块与模型。让其也可以访问视图的数据。

![QMLApp](assets/2024-06-08/QML App.png)
_MVC设计结构_

## 那Model要怎么设置View的数据呢？

在 `main.cpp` 里面有一个魔法道具： `engine`。他会帮助 `.qml` 文件找到一些变量：

```cpp
QQml ApplicationEngine engine;
```

假如我希望让**所有** QML 都能设置成同一个变量 `SCREEN_WIDTH` ，那么我可以在 `main.cpp` 中设置一个**全局上下文对象**：

```cpp
// 其他 include
#include <QQmlContext>

int main() {
    // other codes
    QQmlContext* context = engine.rootContext();
    context->setContextProperty("SCREEN_WIDTH", 200);
    // other codes
}
```

此时任何QML类型都可以使用该变量，其值为 `200`。

```qml
Window {
    width: SCREEN_WIDTH
    // ...
}
```

学会以后就可以随意设置 `200` 的值啦！

### 如何私有化一个数据？

当你使用 `.qml` 文件定义了一个类型，但你希望其可以类似 C++ 的类一样，拥有一些私有 (`private`) 属性，应该怎么操作？

```qml
import QtQuick 2.0
import QtQuick.Controls 2.5

Rectangle {
    width: 200
    height: 100
    color: "black"
    property int privateValue: 0
}
```

假如你这样写，那么外部将会正常访问到 `privateValue` 这个值。

解决方案就是使用 `QtObject` :

```qml
Rectangle {
    // width:...
    QtObject {
        id: attributes
        property int privateValue: 0
    }
}
```

此时你在其内部可以正常使用 `attributes.privateValue` 访问其值。

如果你希望向外部提供接口，则可以使用别名 (`alias`)：

```qml
Rectangle {
    // width...
    property alias attr: attributes
    QtObject {
        id: attributes
        property int privateValue: 0
    }
}
```

这样其他地方就使用 `attr.privateValue` 访问其内部的值了。

## 如何自己定义一个 QtObject 类？

假如你希望自己定义一个不一样的类，比如 `CharacterData` 用于包含数据库中的角色信息。那么我们需要怎么定义一个这样的类呢？

1. 创建一个 C++ 的类：

![create class](assets/2024-06-08/qt_new.png)

2. 让他继承自 `QObject` 的类，这样他才能在 QML 中使用和访问：

![set class](assets/2024-06-08/qt_class_setting.png)

3. 此时打开 `characterdata.h` 即可开始设置各种属性：

```cpp
// ...
public:
    explicit CharacterData(QObject *parent = nullptr);
    int getId() {
        return m_id;
    }
    void setID();
    // other public things
private:
    int m_id;
    QString m_name_en;
    // other private things
```

### 定义一些基本功能有没有简单方法？

如果你想在外部对该类里面的的各种数据进行**读写操作**，那么上方的写法是可以接受的。但是 Qt Creator 可以帮我们实现更好的写法：

将鼠标移到 `m_id` 上并单击，然后按下 `alt + enter`，选择第一个按下 `enter`：

![shortcut](assets/2024-06-08/shortcut.png)

此时 `characterdata.cpp` 中自动创建了两个函数：

```cpp
int CharacterData::id() const
{
    return m_id;
}

void CharacterData::setId(int newId)
{
    if (m_id == newId)
        return;
    m_id = newId;
    emit idChanged();
}

```

第一个函数 `id()` 比较简单，直接返回当前 `id` 的值；

第二个函数 `setId()` 则可以设置 `id` 的值：如果相等则返回；如果不等则设置为新id并发射一个*信号* `idChanged()`。

> **信号与插槽** 将在下一讲中讲解。
{: .prompt-info }

回到 `characterdata.h` 文件：

```cpp
public:
    // other things
    int id() const;
    void setId(int newId);

private:
    // other things
    Q_PROPERTY(int id READ id WRITE setId NOTIFY idChanged FINAL)
```

`public` 部分中定义了两个函数很好理解。来看看 `private` 部分多了什么：

`Q_PROPERTY` 定义了一个*宏*。宏是什么我也不太清楚。但是这一句声明了一个名称 `id` ，之后在 QML 中就可以直接对这个 `id` 进行读写，而不用调用具体的函数。举例如下：

```qml
CharacterData {
    id: 1001
}
```

这样就可以直接对这个 `CharacterData` 的属性 `id` 自动调用 `WRITE` 对应的函数 `setId` 进行写入操作。

相应的，当 `id` 值被修改时，也会自动执行 `idChanged()` 函数发送信号。

### 如何在QML中访问新创建的类呢？

好问题！还记得上面我们讲过的部分吗？

```cpp
context->setContextProperty("SCREEN_WIDTH", 200);
```

如果值比较简单，使用上下文注册完全没问题。而对于比较复杂的类型，则可以使用**注册类型**的方式进行全局调用。有一个重要的函数可以实现这一功能：

```cpp
qmlRegisterType<CharacterData>("Characters", versionMajor, versionMinor, "CharacterData");
```

首先，`qmlRegisterType` 是一个模板函数，尖括号 `<>` 里面的 `CharacterData` 指定了数据类型；

后面的第一个参数是 QML 文件中导入的文件名称，类似域名；第二个和第三个参数则是主版本号和次版本号。例如：

```cpp
qmlRegisterType<CharacterData>("Characters", 1, 0, "CharacterData");
```

那么此时 QML 的 `import` 这样写：

```qml
import Characters 1.0
```

最后一个参数是类型名称，注册后，可以在 QML 文件中通过这个名称创建和使用该类型的实例。。

> 注意，最后一个参数的首字母一定要大写！
{: .prompt-warning}