---
layout: post
title: 补间动画不播放的问题
description: ""
category: program
analytics: true
tags: [FLASH, AS3]
---

在fla文件中，使用一个原件（名称"a"），制作一段补间动画。
给带有动画的原件"a"，起好实例名称（比如"\_a"）。原件做成导出类。编译成swf文件。
然后再FlashBuilder的代码中，加载swf后，反序列化后，取出导出类。
实例化后，使用实例名称("\_a")取出开始定义好的原件"a"。

这个时候，只要修改a的位置坐标（即使这个坐标和原始坐标是相同的），那么原来制作的动画，就不再播放了。很奇怪的问题……
