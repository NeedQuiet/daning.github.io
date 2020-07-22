---
layout: post
title: "自定义NSSlider"
date:   2020-7-22
tags: [MacOS]
comments: true
author: 大宁
toc: true
---

见名知意，本文介绍如何自定义NSSlider

<!-- more -->

>NSSlider由2部分组成，外层的`NSSlider`以及内部的`NSSliderCell`。通常我们所说的自定义Slider说的就是自定义`NSSliderCell`。

## 自定义NSSliderCell

以我自定定义的DNSliderCell举例

### 1. 定义/实现代理

>NSSliderCell继承自NSActionCell，NSActionCell继承自NSCell，有关Tracking的方法都在NSCell中，
所以我们完全可以通过重写Tracking的相关方法来自定义Slider的点击、拖拽...

```swift
@objc protocol DNSliderCellDelegate {
    // 我这里定义的都是可选代理，代理者可根据需求去实现方法 
    // 开始拖动
    @objc optional func startTracking(doubleValue:Double ,sender:DNSliderCell)
    // 正在拖动
    @objc optional func continueTracking(doubleValue:Double ,sender:DNSliderCell)
    // 拖动结束 & 点击结束
    @objc optional func stopTracking(doubleValue:Double ,sender:DNSliderCell)
}

extension DNSliderCell {
    //MARK: 开始拖动
    override func startTracking(at startPoint: NSPoint, in controlView: NSView) -> Bool {
        delegate?.startTracking?(doubleValue: doubleValue, sender: self)
        return super.startTracking(at: startPoint, in: controlView)
    }
    //MARK: 拖动中
    override func continueTracking(last lastPoint: NSPoint, current currentPoint: NSPoint, in controlView: NSView) -> Bool {
        delegate?.continueTracking?(doubleValue: doubleValue, sender: self)
        return super.continueTracking(last: lastPoint, current: currentPoint, in: controlView)
    }
    //MARK: 结束拖动 & 点击结束
    override func stopTracking(last lastPoint: NSPoint, current stopPoint: NSPoint, in controlView: NSView, mouseIsUp flag: Bool) {
        delegate?.stopTracking?(doubleValue: doubleValue, sender: self)
        super.stopTracking(last: lastPoint, current: stopPoint, in: controlView, mouseIsUp: flag)
    }
}
```

### 2. 定义属性

>可以根据需求自己定义常量或者变量，`需要注意`的是，Swift5、macOS Catalina 10.15.4上，xib创建的NSSlider会向两边各延伸2个px（其他版本是否如此不确定），例如我自己定义长度1000的slider，实际上长度为1004，所以定义了个Offset方便计算。

```swift
class DNSliderCell: NSSliderCell {
    let Offset:CGFloat = 2 // 进度条，实际上向（根据slider方向）左/右 或者 上/下 偏移2
    let progressColor = kRedHighlightColor // 进度条进度背景色
    var backgroundColor = NSColor.clear // 进度条整体背景色
    let knobColor = kRedHighlightColor // 按钮颜色
    let sliderThickness:CGFloat = 3.0 // 进度条厚度
    let sliderBarRadius:CGFloat = 0 // 进度条圆角
    let KnobWidth:CGFloat = 12.0 // 按钮宽度
    let KnobHeight:CGFloat = 12.0 // 按钮高度
    var progressRect = NSRect() // 进度的Rect
    var needControlKnobHidden = false // 是否需要控制旋钮显示隐藏
    
    weak var delegate:DNSliderCellDelegate? // 代理
    
    // slider是否是竖直
    lazy var isSliderVertical:Bool = {
        let isVertical = (controlView as? DNSlider)?.isVertical == true
        return isVertical
    }()
}
```

### 3. 重绘slider

> 要根据其Slider是横着还是竖着的来计算不同的rect

```swift
extension DNSliderCell {
    //MARK: - 重绘slider
    override func drawBar(inside rect: NSRect, flipped: Bool) {
        //MARK: 修改进度条'整体'背景色
        // 1. 获取 slider 的 rect
        var sliderRect = rect
        // 2. 更改 slider 厚度
        if isSliderVertical {
            sliderRect.size.width = sliderThickness
        } else {
            sliderRect.size.height = sliderThickness
        }
        // 3. 填充背景色
        let background = NSBezierPath(roundedRect: sliderRect, xRadius: sliderBarRadius, yRadius: sliderBarRadius)
        self.backgroundColor.setFill()
        background.fill()
        
        //MARK: 修改进度条'进度'背景色
        // 1. 计算当前进度的占有比例
        let value:CGFloat = CGFloat((doubleValue - minValue) / (maxValue - minValue))
        // 2. 根据比例计算进度应有的长度
        var viewLength:CGFloat
        if isSliderVertical {
            viewLength = (controlView?.frame.size.height)!
        } else {
            viewLength = (controlView?.frame.size.width)!
        }
 
        let finalLength:CGFloat = value * (viewLength - Offset * 2)
        // 3. 计算进度的Rect
        progressRect = sliderRect
        if isSliderVertical {
            progressRect.size.height = finalLength
            progressRect.origin.y = Offset
        } else {
            progressRect.size.width = finalLength
            progressRect.origin.x = Offset
        }
        
        // 4. 填充进度背景色
        let active = NSBezierPath(roundedRect: progressRect, xRadius: sliderBarRadius, yRadius: sliderBarRadius)
        self.progressColor.setFill()
        active.fill()
    }
}
```

### 4. 重绘旋钮

> 要根据其Slider是横着还是竖着的来计算不同的rect

```swift
//MARK: - 重绘旋钮
    override func drawKnob() {
        // 根据比例计算进度应有的OriginX
        let value:CGFloat = CGFloat((doubleValue - minValue) / (maxValue - minValue))
        var viewLength:CGFloat
        if isSliderVertical{
            viewLength = (controlView?.frame.size.height)!
        } else {
            viewLength = (controlView?.frame.size.width)!
        }
        
        var customKnobRect:NSRect
        
        if isSliderVertical {
            //                   Offset +  进度  * ((       实际宽度     ) -  knob宽度 )
            let finalStart:CGFloat = Offset + value * (viewLength - Offset * 2 - KnobHeight)
            // 计算Knob的Rect
            customKnobRect = NSRect(x: progressRect.origin.x + progressRect.size.width / 2 - KnobWidth / 2, y: finalStart, width: KnobWidth , height: KnobHeight)
        } else {
            //                   Offset +  进度  * ((       实际宽度      ) -  knob宽度 )
            let finalStart:CGFloat = Offset + value * (viewLength - Offset * 2 - KnobWidth)
            // 计算Knob的Rect
            customKnobRect = NSRect(x: finalStart, y: progressRect.origin.y + progressRect.size.height / 2 - KnobHeight / 2, width: KnobWidth , height: KnobHeight)
        }
        
        
        // 填充Knob的颜色
        let background = NSBezierPath(roundedRect: customKnobRect, xRadius: KnobWidth / 2, yRadius: KnobHeight / 2)
        if needControlKnobHidden == true {
            if (controlView as? DNSlider)?.shouKnob == true {
                self.knobColor.setFill()
            } else {
                NSColor.clear.setFill()
            }
        } else {
            self.knobColor.setFill()
        }

        background.fill()
    }
```

## 自定义NSSlider

> 自定义NSSlider，可以处理鼠标滑过时鼠标指针的状态、控制knob的显示隐藏等; 不废话直接上代码！

```swift
class DNSlider: NSSlider {
    private var trackingArea:NSTrackingArea?
    // 是否显示knob
    var shouKnob:Bool = false    
}

extension DNSlider {
    override func updateTrackingAreas() {
        super.updateTrackingAreas()
        if trackingArea != nil {
            self.removeTrackingArea(trackingArea!)
        }
        
        // 将设置追踪区域为控件大小
        // 设置鼠标追踪区域，如果不设置追踪区域，mouseEntered和mouseExited会无效
        trackingArea = NSTrackingArea(rect: bounds, options: [.mouseEnteredAndExited, .activeAlways], owner: self, userInfo: nil)
        self.addTrackingArea(trackingArea!)
    }
    
    override func mouseEntered(with event: NSEvent) {
        super.mouseEntered(with: event)
        NSCursor.pointingHand.set()
        /*
            通过设置 isHighlighted 取反来触发 cell 的 drawKnob 方法;
            通过 shouKnob 来控制是否显示(因为isHighlighted受其他状态影响，无法精准控制)
         */
        shouKnob = true
        isHighlighted = !isHighlighted
    }
    
    override func mouseExited(with event: NSEvent) {
        super.mouseExited(with: event)
        NSCursor.arrow.set()
        shouKnob = false
        
        // 不直接设置true/false是因为如果在mouseExited前isHighlighted已经是true/false，那就无法触发cell重新渲染了
        isHighlighted = !isHighlighted
    }
    // 点击后，状态会刷新，此时如不做更改会默认改回指针状态
    override func cursorUpdate(with event: NSEvent) {
        super.cursorUpdate(with: event)
        NSCursor.pointingHand.set()
    }
}
```