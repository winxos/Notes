# qt 相关内容

> wvv 20220114

### windows 下环境搭建



### Ubuntu下环境搭建

可以自己到官网下载，也可以简单的通过apt安装

```shell
sudo apt install qt5-default qtcreator
```

下面命令用于安装帮助文件和示例工程

```shell
sudo apt install qt5-doc qt5-doc-html qtbase5-doc-html qtbase5-examples
```



### qml使用

### qml 调用cpp

#### 方法一 利用Q_INVOKABLE

1. 新建test 类

```cpp
#ifndef TEST_H
#define TEST_H

#include <QObject>
#include <QDebug>
class test:public QObject
{
    Q_OBJECT
public:
    using QObject::QObject;
    test();
    Q_INVOKABLE void btn_click()
    {
    qDebug()<<"click";
    }
};

#endif // TEST_H
```
核心在于继承于QObject，调用Q_OBJECT宏，方法加上 **Q_INVOKABLE**

2. 修改main
```cpp
#include <QGuiApplication>
#include <QQmlApplicationEngine>
#include <test.h>
#include <QQmlContext>
int main(int argc, char *argv[])
{
    test t;
    QCoreApplication::setAttribute(Qt::AA_EnableHighDpiScaling);

    QGuiApplication app(argc, argv);
    QQmlApplicationEngine engine;
    const QUrl url(QStringLiteral("qrc:/main.qml"));
    QObject::connect(&engine, &QQmlApplicationEngine::objectCreated,
                     &app, [url](QObject *obj, const QUrl &objUrl) {
        if (!obj && url == objUrl)
            QCoreApplication::exit(-1);
    }, Qt::QueuedConnection);
    engine.load(url);
    auto cc = engine.rootContext();
    cc->setContextProperty("test",&t);
    return app.exec();
}
```
核心在于 **setContextProperty**

3. 修改qml文件

以btn为例

```json
import QtQuick 2.14
import QtQuick.Window 2.14
import QtQuick.Controls 2.13
Window {
    visible: true
    width: 640
    height: 480
    title: qsTr("Hello World")
    Button {
        id: button
        x: 346
        y: 121
        width: 239
        height: 84
        text: qsTr("Button")
        font.pointSize: 30
    }

    Connections {
        target: button
        onClicked: test.btn_click()
    }
}
```

#### 方法二 利用slots

1. 新建test 类

```cpp
#ifndef TEST_H
#define TEST_H

#include <QObject>
#include <QDebug>
class test:public QObject
{
    Q_OBJECT
public:
    test();
    public slots:
    void click2()
    {
qDebug()<<"click2";
    }
};

#endif // TEST_H
```
核心在于继承于QObject，调用Q_OBJECT宏，方法public slots 修饰
2. 修改main

```cpp
#include <QGuiApplication>
#include <QQmlApplicationEngine>
#include <test.h>
int main(int argc, char *argv[])
{
    QCoreApplication::setAttribute(Qt::AA_EnableHighDpiScaling);

    QGuiApplication app(argc, argv);
    qmlRegisterType<test>("wvv.Test",1,0,"Test");
    QQmlApplicationEngine engine;
    const QUrl url(QStringLiteral("qrc:/main.qml"));
    QObject::connect(&engine, &QQmlApplicationEngine::objectCreated,
                     &app, [url](QObject *obj, const QUrl &objUrl) {
        if (!obj && url == objUrl)
            QCoreApplication::exit(-1);
    }, Qt::QueuedConnection);
    engine.load(url);
    return app.exec();
}
```
核心在于用qmlRegisterType注册，第一个参数是qml里面import用的，第4个参数是qml中对象名称
3. 修改qml

```cpp
import QtQuick 2.14
import QtQuick.Window 2.14
import QtQuick.Controls 2.13
import wvv.Test 1.0
Window {
    visible: true
    width: 640
    height: 480
    title: qsTr("Hello World")
    Test{
        id:ttt;
    }
    Button {
        id: button1
        x: 72
        y: 159
        text: qsTr("Button")
    }
    Connections {
        target: button1
        onClicked: ttt.click2()
    }
}
```
核心在于import main中注册的对象，然后示例Test对象，就可以通过信号连接了

#### 方法比较

第二种方法示例在qml中，main修改少，解耦更好一些。
