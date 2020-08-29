---
layout: post
title: "Mac监听键盘点击"
date:   2020-8-28
tags: [MacOS]
comments: true
author: 大宁
toc: true
---

概述：<br>
总结了Swift中，暂时发现的3种键盘(鼠标)点击的监听方式

<!-- more -->

>注意，以下方法中，涉及到全局键盘监听的都需要系统提供权限才能正常使用，判断方式：
```swift
//系统首选项>安全&隐私权>隐私权>辅助功能 - 请确保您的应用已被选中。
print("辅助功能:",AXIsProcessTrusted() ? "信任" : "不信任")
```

# 一、系统自带的API
>其实这两个监听就可以满足大部分的需求了，唯一的不足是像F7~F9的媒体功能键无法监听

- Global监听
```swift
    NSEvent.addGlobalMonitorForEvents(matching: [.keyDown]) { [unowned self] (event) in
        // flags貌似是获取与常规键无关的按键
        let flags = event.modifierFlags.intersection(.deviceIndependentFlagsMask)
        // 常规 option || 带有左右键时的 cmd+option
        if (flags == NSEvent.ModifierFlags.option || flags.contains([.command,.option])) {
            self.playControlWithKeyCode(event.keyCode)
        }
    }
```

- Local监听
```swift
    NSEvent.addLocalMonitorForEvents(matching: [.keyDown]) { [unowned self] (event) -> NSEvent?  in
        if self.TextFieldIsEditing == true { return event}
        let code = event.keyCode
        let flags = event.modifierFlags.intersection(.deviceIndependentFlagsMask)
        
        if (flags.contains([.option])) { // 包含option
            self.playControlWithKeyCode(code)
        } else {
            if flags == NSEvent.ModifierFlags.command {
                if code == kVK_ANSI_W {
                    WindowManager.share.currentWindow?.close()
                }
            } else {
                if (code == kVK_ANSI_W || code == kVK_Space){
                    self.playControlWithKeyCode(code)
                }
            }
        }
        return nil
    }
```

# 二、子类化NSApplication
我们可以通过子类化NSApplication，拦截sendEvent来实现获取键盘鼠标的点击事件，就像注释中描述的，虽然可以达到获取媒体键的点击，但是却无法拦截唤起的iTunes，不过用来获取其他按钮的点击事件也是可以的

>注意：子类化NSApplication，需更改Info.plist里的NSPrincipalClass，如果是Swift则设置为$(PRODUCT_MODULE_NAME).MyApplication才会有效，如果是OC则直接设置为MyApplication即可，当然类名可以自己随意定义，我这里仅仅举个例子

```swift
class MyApplication: NSApplication {
    // 可以通过重写sendEvent监听媒体功能键，但是会唤起iTunes音乐，无法拦截，所以放弃
    override func sendEvent(_ event: NSEvent) {
        if  event.type == .systemDefined &&
            event.subtype == .screenChanged
        {
            let keyCode : Int32 = (Int32((event.data1 & 0xFFFF0000) >> 16))
            let keyFlags = (event.data1 & 0x0000FFFF)
            let keyState = ((keyFlags & 0xFF00) >> 8) == 0xA

            mediaKeyEvent(withKeyCode: keyCode, andState: keyState)
            return
        }
        super.sendEvent(event)
    }
}
```

# 三、CGEvent.tapCreate拦截
>此方法一样可以直接获取键盘事件，比子类化NSApplication高级的是可以拦截事件，例如拦截媒体键对iTunes的控制，不废话，直接上代码
>>同时，有趣的是这种方式，在首次加载时，会弹出系统提示框，获取隐私服务权限，比较友好

```swift
// 这个方法也可以拿来监听键盘其他key的点击
    func startListenCGEventTap() {
        func myCGEventCallback(proxy: CGEventTapProxy, type: CGEventType, event: CGEvent, refcon: UnsafeMutableRawPointer?) -> Unmanaged<CGEvent>? {
            if(type == .tapDisabledByTimeout){
                return Unmanaged.passRetained(event)
            }
            
            if let nsEvent = NSEvent.init(cgEvent: event) {
                if (nsEvent.type == .systemDefined && nsEvent.subtype == .screenChanged ) {
                    let keyCode : Int32 = (Int32((nsEvent.data1 & 0xFFFF0000) >> 16))
                    let keyFlags = (nsEvent.data1 & 0x0000FFFF)
                    let keyState = ((keyFlags & 0xFF00) >> 8) == 0xA
//                    let keyIsRepeat = (keyFlags & 0x1) > 0
//                    if keyIsRepeat {
//                        print("keyIsRepeat")
//                        return nil
//                    }
                    mediaKeyEvent(withKeyCode: keyCode, andState: keyState)
                    return nil
                }
            }

            return Unmanaged.passRetained(event)
        }
        
        // let eventMask = (1 << CGEventType.keyDown.rawValue) | (1 << CGEventType.keyUp.rawValue | (1 << CGEventType.flagsChanged.rawValue))
        
        // 创建eventTap，此时会唤起’打开隐私权限‘弹窗
        if let eventTap = CGEvent.tapCreate(tap: .cgSessionEventTap,
                                            place: .headInsertEventTap,
                                            options: .defaultTap,
                                            eventsOfInterest: CGEventMask(NX_SYSDEFINEDMASK),
                                            callback: myCGEventCallback,
                                            userInfo: nil) {
            let runLoopSource = CFMachPortCreateRunLoopSource(kCFAllocatorSystemDefault, eventTap, 0)
            CFRunLoopAddSource(CFRunLoopGetCurrent(), runLoopSource, .commonModes)
            CGEvent.tapEnable(tap: eventTap, enable: true)
            CFRunLoopRun()
        } else {
            print("failed to create event tap")
//            exit(1)
        }
    }
```