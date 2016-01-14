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
打开本章的配套的工程VacationSpots，在iPhone 6模拟器上运行，能够看到APP有一些UI的问题，不要担心，在后面将会修复这些问题。简单梳理一下问题如下：
1. 图中标出的内容没有在垂直方向居中
![](/images/2016.01.13/01.png)  

2. 点击列表中的London Cell进入详情页面，最下面的三个按钮没有平均分配空间：
![](/images/2016.01.13/02.png)  

3. 点击WEATHER旁边的hide按钮，内容是被隐藏了，但是留下了一块空白，下面的内容没有移动上来：
![](/images/2016.01.13/03.png)    

4. WHAT TO SEE 部分在WHY VISIT的下面会更加合理一点。  

现在已经了解了这些问题，下面开始用UIStackView来修改这些问题。打开Main.storyboard，查看如下的Controller scene：
![](/images/2016.01.13/04.png)    
>能够注意到上面的每个对应的控件都有背景颜色，这个只是为了帮助我们查看这些属性的变化。这些背景颜色在运行的时候都会被去掉，通过如下代码，如果你想让它们在运行的时候也显示注释掉这些代码就可以：
{% codeblock lang:swift %}
// SpotInfoViewController.swift
 override func viewDidLoad() {
    super.viewDidLoad()

    // Clear background colors from labels and buttons
    for view in backgroundColoredViews {
      view.backgroundColor = UIColor.clearColor()
    }

    ........
  }
{% endcodeblock %}    

在Storyboard中的的控件都通过outlet与SpotInfoViewController.swift中对应的属性进行了关联。在storyboard中显示的名字对应SpotInfoViewController.swift对应的变量。
{% codeblock lang:swift %}
{% endcodeblock %} 
{% codeblock lang:swift %}
{% endcodeblock %} 
{% codeblock lang:swift %}
{% endcodeblock %} 
{% codeblock lang:swift %}
{% endcodeblock %} 
{% codeblock lang:swift %}
{% endcodeblock %} 
{% codeblock lang:swift %}
{% endcodeblock %} 
{% codeblock lang:swift %}
{% endcodeblock %} 
{% codeblock lang:swift %}
{% endcodeblock %} 
{% codeblock lang:swift %}
{% endcodeblock %} 
{% codeblock lang:swift %}
{% endcodeblock %} 
{% codeblock lang:swift %}
{% endcodeblock %} 

