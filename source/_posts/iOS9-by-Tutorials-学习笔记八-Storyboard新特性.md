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

<!--- more --->

现在我们在Xcode7中能够通过storyboard reference解决这个问题了。它能够很容易的将storyboard分割成为更小的storyboard，并且能够直接在storyboard中使用segue。更多的小的storyboard也能让团队的成员更加独立的工作，而不会影响到其他的团队成员。  

#### Creating your first storyboard reference   
目前的情况，Prepped是一个很小的app，但是同样也可以在结构上进行storyboard的划分。Container view controller就是一个很好的从功能上划分的例子。   

Prepped使用的tab bar controller，在这种情况下，将每个tab的子view controller划分进一个storyboard是一个不错的选择（当然每个tab bar子VC下还可以划分）。    

打开Main.storyboard,双击空白区域，缩小storyboard区域，能够看到所有的scene。选中除tab bar controller的所有的scene。
![](/images/2016.01.25/01.png)

选择 **Editor\Refactor to Storyboard**，输入 **Checklists.storyboard** 作为新的storyboard的名字。点击Save。这样我们就将一部分的Scene拆分到了新的storyboard中。Main.storyboard也变成了下面的样子：
![](/images/2016.01.25/02.png)  

选中图中的Reference，查看右边的属性检查器，能够看到Storyboard和Referenced ID，storyboard根据这两个属性去确定该reference真正指向的Scene。双加该reference也能够跳转到真正的Scene。如果我们删除了上图中的Referenced ID，默认情况下指向的是reference Storyboard中Initial View Controller。
![](/images/2016.01.25/03.png)  

#### Storyboards within a team  
由于在团队中都惧怕Storyboard的冲突，所以还是很少使用。但是现在storyboard reference能够避免这些冲突。我们可能都遇到过这种情况，我们在一个storyboard钟创建了一个view controller，在另一个storyboard想要跳转到它，一般这种情况我们都是使用代码跳转了，现在我们可以使用storyboard实现了。
> 添加工程目录内的Diary目录到工程中来  

打开Main.storyboard，拽一个storyboard reference添加到storyboard的空白区域：
![](/images/2016.01.25/04.png)    

运行程序，显示如下：
![](/images/2016.01.25/05.png)   

### Views in the scene dock
Scene Dock是在Storyboard Scene顶部的一个区域（下图中有标注），我们可以将View添加到scene dock上，添加到scene dock上的view不会被初始化添加到view controller的view的subviews中，但是你能够使用IBOutlet引用使用它。按照如图添加然后设置该View的background color为#FFFAE8.
![](/images/2016.01.25/06.png)    

按照下图，按住ctrl连接，在弹出菜单中选择selectedBackgroundView：
![](/images/2016.01.25/07.png)     

运行程序，发现我们选中的item上面添加了背景。  

上面的例子中我们只是通过storyboard使用了scene dock中的view，我们也可以使用IBOutlet引用到我们的代码中，然后由代码控制在什么情况下的隐藏与显示。

### Using multiple bar buttons
在Xcode 7中我们可以在storyboard中，为view controller的Navigation bar在同一侧添加多个按钮了。  

打开ChecklistDetail.storyboard，选中 Checklist Detail View Controller，从Object Library中拽多个bar button item添加到右侧的Navigation bar上，这样就添加了多个bar button，添加完后其他的使用方式就跟原来的基本一样了，这里就不重复了。
![](/images/2016.01.25/08.png)    

最近一直在比较累，所以这篇整理的比较简单，只是挑选了一些重点的写出来了，如果哪里感觉不是太详细，还请去看英文的原版。
