---
layout: post
title: "MarkDown"
date:   2020-1-10
tags: [markdown]
comments: true
author: 大宁
toc: true
---

#   MarkDown
概述：简单记录一下vscode上使用markdown all in one插件后，简单的markdown写法

目录：F1后Create Table of Content即可创建目录，看样子是根据标题来自动同步Table
- [MarkDown](#markdown)
  - [1.标题](#1%e6%a0%87%e9%a2%98)
  - [2.锚点](#2%e9%94%9a%e7%82%b9)
  - [3.代码块](#3%e4%bb%a3%e7%a0%81%e5%9d%97)
  - [4.菜单](#4%e8%8f%9c%e5%8d%95)
  - [5.常用语法](#5%e5%b8%b8%e7%94%a8%e8%af%ad%e6%b3%95)
  - [6.常用快捷键](#6%e5%b8%b8%e7%94%a8%e5%bf%ab%e6%8d%b7%e9%94%ae)

## 1.标题
  - 控制标题大小快捷键
    - 变大 shift + ctrl + [ 
    - 缩小 shift + ctrl + ]
## 2.锚点
可以利用html的a标签，进行锚点跳转，例：A、B互跳
  ><a id="text" href="#sssss">锚点A</a>

  ><a id="sssss" href="#text">锚点B</a>
  ```html
  <a id="text" href="#sssss">锚点A</a>
  <a id="sssss" href="#text">锚点B</a>
  ```

## 3.代码块
1. Python示例(```python)
```python
import os
#asdasdasdasd
print('python 代码块？')
```
2. html示例(```html)
```html
<h1>哎哟不错哦</h1>
```
3. OC示例(```ObjectiveC)
```ObjectiveC
int a = 0;
// asdasdasd
NSArray * testArr = @[@"asdasd",@"awww"];
UIlabel * a = [[UILabel alloc]init];
```

## 4.菜单
菜单用-、+、*加空格都可以
+ 加号+ 
* 星号*
- 减号-

## 5.常用语法
- 高亮强调：`就是两个反引号把内容括起来` ``
- 引用符号： `>`
  >用来突出内容
- 链接： 
  - 格式：[标签名字]（#链接地址）
  - [百度](#https://www.baidu.com)
```
[百度](#https://www.baidu.com)
```
- 插入图片
  - 例：![test.png](https://i.loli.net/2020/01/09/n9bcw14OXN82lMD.png)
  - 代码
```
![图片描述](图片链接)
```
  
- 分割线：就是3个星号 ***
  ***
- 插入视频：

<video id="video" controls="" preload="none" poster="https://i.loli.net/2020/01/09/n9bcw14OXN82lMD.png">
  <source id="mp4" src="https://video.pearvideo.com/mp4/third/20200109/cont-1640431-10042874-102056-hd.mp4" type="video/mp4">
</video>

```html
<video id="video" controls="" preload="none" poster="https://i.loli.net/2020/01/09/n9bcw14OXN82lMD.png">
  <source id="mp4" src="https://video.pearvideo.com/mp4/third/20200109/cont-1640431-10042874-102056-hd.mp4" type="video/mp4">
</video>
```

- 插入音频、
  
<audio id="audio" controls="" preload="none">
    <source id="mp3" src="http://qiniu.cloud.fandong.me/Music_iP%E8%B5%B5%E9%9C%B2%20-%20%E7%A6%BB%E6%AD%8C%20%28Live%29.mp3">
</audio>

```html
<audio id="audio" controls="" preload="none">
    <source id="mp3" src="http://qiniu.cloud.fandong.me/Music_iP%E8%B5%B5%E9%9C%B2%20-%20%E7%A6%BB%E6%AD%8C%20%28Live%29.mp3">
</audio>
```

- 表格:

```
| 左对齐 | 居中  | 右对齐 |
| ------ | :---: | -----: |
| 香蕉   |  $1   |      5 |
| 苹果   |  $1   |      6 |
| 草莓   |  $1   |      7 |
```
  
| 左对齐 | 居中  | 右对齐 |
| ------ | :---: | -----: |
| 香蕉   |  $1   |      5 |
| 苹果   |  $1   |      6 |
| 草莓   |  $1   |      7 |

## 6.常用快捷键
- **字体加粗** ：conmmond + b

- *字体斜体*：conmmond + i