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

#### Your first stack view
我们先用Stack View解决我们问题列表中底部按钮的问题。使用UIStackView能够在一个坐标抽上分配位置和控件之间的空间。幸运的是将Views嵌入到UIStackView中并没有太难。在Storyboard中的Spot Info View Controller选择如下三个按钮控件：
![](/images/2016.01.13/05.png)

选择好三个按钮后，在Storyboard中点击如下的按钮：
![](/images/2016.01.13/06.png)

当Views被嵌入UIStackView之后，Views的约束都被移除了，同时需要设置UIStackView的约束。选中UIStackView，然后按照如图添加约束：
![](/images/2016.01.13/07.png)

>这里有个选择UIStackView的技巧，由于UIStackView是在按钮的后面，很不好选中，我们可以按住Shift，右击，在出现的菜单中列出来当前点击位置所有的View，我们可以选择UIstackView。另一种方法我们可以在outline view中选择。   

设置好约束后，能够看到按钮显示如下，第一个按钮被拉伸了，填充满了UIStackView的剩余空间。UIstackView有一个Distribution属性，用于控制Views怎么在UIStackView中显示，现在设置的是Fill，即将会填充满UIStackView。为了这个目的，UIstackView将会根据View的ccontent hugging优先级去拉伸View，最低的将会被拉伸。如果优先级一样，将会拉伸第一个。
![](/images/2016.01.13/08.png)  

我们的目的是让View间的距离相等，在Attributes inspector中修改Distribution为Equal Spacing。
![](/images/2016.01.13/09.png)

运行APP，能够看到我们的按钮显示正确了：
![](/images/2016.01.13/10.png)  

#### 一些思考
思考一下你使用Auto Layout，通过约束实现上面的要求，那将是一种什么令人"愉悦"的行为。可能你很熟悉Auto Layout，认为这些东西都很简单，那么你在考虑一下如果我们后面有需求添加一个按钮，删除一个按钮呢，怎么办呢？约束删除了重新添加吗？如果使用UIStackView，这些将变得比较简单，只要我们添加或者删除View，其他的工作UIstackView就会帮我们做了。    

UISTackView的更加深入的讲解，将会在下一篇文章中继续介绍。这里先介绍一下Auto Layout的新特性：layout anchors和layout guides。

### Layout anchors
Layout anchors提供了我们一种简单的创建约束的方式。  

想象一下我们在iOS 9之前创建一个约束，简直就是天书，但是在iOS 9中使用Layout anchors将会简单好多，下面是两种的对比：
{% codeblock lang:swift %}
// iOS9以前
let constraint = NSLayoutConstraint(item: topLabel, attribute: .Bottom, relatedBy: .Equal, toItem: bottomLabel, attribute: .Top, multiplier: 1, constant: 8)

// iOS 9
let constraint = topLabel.bottomAnchor.constraintEqualToAnchor(bottomLabel.topAnchor, constant: 8)
{% endcodeblock %}   

Layout anchors不仅理解起来简单，而且写起来也简单了。   

对应于我们在iOS 9以前添加约束时候的attribute，基本都有与之对应的anchor，例如top对应topAnchor，bottom对应bottomAnchor等。
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
