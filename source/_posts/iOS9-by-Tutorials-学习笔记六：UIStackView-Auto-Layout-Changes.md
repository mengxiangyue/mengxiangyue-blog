title: 'iOS9 by Tutorials 学习笔记六：UIStackView & Auto Layout Changes'
date: 2016-01-13 22:20:34
categories:
  - iOS 9 by Tutoials
tags:
  - iOS 9
---

今天这是第六篇笔记，现在回过头去看，我也没有想到自己能够更新到第六篇。我算是一个比较懒的人了，现在已经不太喜欢动手敲代码了。在写这几篇笔记的时候，我需要一边看英文的文档，一边测试代码，还得考虑怎么能够写明白。这里有点说明，我的英语水平只是四级，语文水平只能用呵呵评价了，文章中的语句难免会有不通顺的地方，希望能够把语义表述清楚。    

闲话多了，回到正题，这篇文章介绍UIStackView和一些Auto Layout的改变。   

UIStackView我个人理解是为了解决使用Storyboard添加的约束需要经常变化的情况。我想我们可能都在开发中遇到过修改约束的情况，一般是把约束与一个outlet的约束link起来，然后代码修改，但是这个操作起来是不方便的。UIStackView通过修改一些简单的属性，例如alignment, distribution, and spacing，从而让UIStackView根据我们的修改自动调整内部的显示。    

Auto Layout的改变主要是介绍layout anchors和layout guides。

<!---more--->

### Getting Started
