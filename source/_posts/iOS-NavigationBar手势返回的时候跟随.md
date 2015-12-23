title: iOS NavigationBar手势返回的时候跟随
date: 2015-12-23 23:22:39
categories:
  - iOS Tips
tags:
---

在iOS的开发中有时候会遇到这样的情况，在开发的过程中有两个界面，这两个界面使用UINavigationController串联起来，然后在第一个ViewController中不显示UINavigationBar，在第二个显示UINavigationBar。iOS在手势返回的时候默认情况下iOS的NavigationBar是固定的，然后再去做一些渐变位移等动画，但是如果我们在一个界面有NavigationBar，一个没有这样的动画就会很难看。这时候我们希望第二个界面手势返回的时候，NavigationBar跟着界面一起移动。 实现类似下面的效果:   

![](/images/2015.12.23.NavigationBarTest.gif)  

<!--more-->

这个效果中有两个ViewController，在第二个Controller手势返回的时候，UINavigationBar是跟随着Controller的。如果只是在第一个界面的时候使用这种效果比较简单，只要在两个Controller中添加如下代码即可：

{% codeblock lang:swift %}  
// 第一个ViewController
override func viewWillAppear(animated: Bool) {
    super.viewWillAppear(animated)
    // 这里一定要使用这个方法 否则会有问题
    self.navigationController?.setNavigationBarHidden(true, animated: true)
}

// 第二个ViewController
override func viewWillAppear(animated: Bool) {
    super.viewWillAppear(animated)
    self.navigationController?.setNavigationBarHidden(false, animated: true)
}
{% endcodeblock %}

上面的代码只能实现第一个UINavigationController第一个ViewController隐藏，第二个显示的时候实现跟随效果。如果想实现所有的界面都有跟随的效果，那需要自定义ViewController的转场动画，网上有人已经实现了这个效果，感兴趣的可以下载看一下，地址：<https://github.com/esonchen/CCSlideNavigationTransition>。   

这篇文章比较短，只是一个简单的Tip而已。
