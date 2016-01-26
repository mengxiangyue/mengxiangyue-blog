title: 'iOS9 by Tutorials 学习笔记八:Storyboard新特性'
date: 2016-01-25 23:06:34
categories:
  - iOS 9 by Tutoials
tags:
  - iOS 9
---

终于整理到第八篇了，中途好几次都想放弃了，最终还是坚持下来了，我自己的计划应该除了这篇还有最多三篇就结束了吧。   

回到正题，苹果最近几点一直在努力推行Storyboard，每年的WWDC都会针对Storyboard有一定的增强，今年也不例外，Storyboard添加了一些新的特性：
1. 可以使用storyboard references将一个Scene链接到另一个storyboard中
2. 使用scene dock给view controller添加附加的view
3. 在navigation bar上添加多个button

### Getting started
打开本节配套的工程，在模拟器上运行一下，看看都有什么功能。下面看一下代码 **ChecklistsViewController.swift** 显示checklist，**ChecklistDetailViewController.swift** 显示list中的每一个item，**Main.storyboard** 包含用户界面。这个App并没有完成，在本节中将会添加一些功能，完善该APP。  

### Storyboard references
如果你在一个大的项目中使用了storyboard，你将会慢慢感觉storyboard越来越大，会越来越笨重，并且还经常会发生冲突。这些导致了storyboard慢慢丢失了它的优势。   

你可能已经通过将Scene分别放到多个storyboard中的方式解决了一部分的问题，但是同时也遇到了一些问题：在两个Scene之间的segue不能直接添加了，显示一个View Controller有时候必须通过代码创建显示。   

现在我们在Xcode7中能够通过storyboard reference解决这个问题了。它能够很容易的将storyboard分割成为更小的storyboard，并且能够直接在storyboard中使用segue。更多的小的storyboard也能让团队的成员更加独立的工作，而不会影响到其他的团队成员。  

#### Creating your first storyboard reference   
目前的情况，Prepped是一个很小的app，但是同样也可以在结构上进行storyboard的划分。Container view controller就是一个很好的从功能上划分的例子。   

Prepped使用的tab bar controller，在这种情况下，将每个tab的子view controller划分进一个storyboard是一个不错的选择（当然每个tab bar子VC下还可以划分）。    

打开Main.storyboard,双击空白区域，缩小storyboard区域，能够看到所有的scene。选中除tab bar controller的所有的scene。
![](/images/2016.01.25/01.png)
