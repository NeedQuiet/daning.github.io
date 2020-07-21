---
layout: post
title: "MacOS关闭按钮与Reopen"
date:   2020-7-21
tags: [MacOS]
comments: true
author: 大宁
toc: false
---

概述：<br>
在点击Window上的Close按钮后，再在dock栏里重新选中软件，这时软件的各种状态都会重置，一些设置的unowned的地方甚至会crash，这相当于一个半成品的reopen；`我们需要的是点击关闭按钮并不会杀掉软件，而是隐藏起来，点击后展示之前软件的显示状态`；

<!-- more -->

## 步骤

实现方式很简单，一共就需要两步

```swift
//1. 首先定义mainWindow，并在加载后赋值
var mainWindow: NSWindow!
    func applicationDidFinishLaunching(_ aNotification: Notification) {
        mainWindow = NSApplication.shared.windows[0]
    }
```

```swift
//2. 然后继承NSWindowDelegate，并实现如下方法
extension AppDelegate: NSWindowDelegate {
    func applicationShouldHandleReopen(_ sender: NSApplication, hasVisibleWindows flag: Bool) -> Bool {
        if !flag {
            mainWindow.makeKeyAndOrderFront(nil) // 这里是核心代码
        }
        return false // 这里要设为false，不然会reopen一个窗口
    }
}
```