title: iOS9-by-Tutorials-学习笔记七：UIStackView 进阶
date: 2016-01-21 23:40:43
categories:
  - iOS 9 by Tutoials
tags:
  - iOS 9
---    

继续接着上篇文章介绍UIStackView，这一篇将涉及到一些稍微深一点的知识点。

### Getting started
首先回顾一下我们计划完善SpotInfoViewController需要完成的任务情况：
1. 底部按钮间距不相同。 完成。
2. 点击Hide按钮，底部的内容应该填充空白位置。 未完成。
3. 交换what to see和weather两部分的内容。 未完成。
4. 在landscape模式下增大底部按钮与底部边缘的距离。 未完成（回头校验）

### Converting the sections   
在完成上面的任务之前，你需要将SpotInfoViewController中的所有的sections转换为使用stack view。在下面的内容中能够学到一下配置stack view的属性，例如alignment、distribution和spacing。

#### Rating section
