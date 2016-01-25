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
4. weather详情隐藏的时候，rating部分纵向显示。

### Converting the sections   
在完成上面的任务之前，你需要将SpotInfoViewController中的所有的sections转换为使用stack view。在下面的内容中能够学到一下配置stack view的属性，例如alignment、distribution和spacing。

#### Rating section

<!--- more --->   
Rating 部分是很容易修改的，只要把它们嵌入到一个stack view就可以了。  

打开Main.storyboard,在Spot Info View Controller场景中选中RATING Lable和stars Label：
![](/images/2016.01.21/01.png)

选中后点击图片右下角指出来的Stack按钮。按照如图设置stack view的约束：
![](/images/2016.01.21/02.png)    

设置stack view的spacing属性为8，storyboard中显示如下，看起来是错误的，我们只需要修改属性distribution为其他值，然后再修改为Fill，就好了。
![](/images/2016.01.21/03.png)  

#### Unembedding a stack view   
有时候我们想要取消一个Stack View，可以选中Stack View，按住Option键点击Stack按钮，在弹出的菜单中选择**Umembed**:
![](/images/2016.01.21/04.png)  

### Your first vertical stack view   
现在你讲创建第一个纵向的stack view。选中WHY VISIT label和<whyVisitLabel> label,点击Stack按钮，Xcode会根据控件的位置创建stack view的正确方向。这两个label嵌入到stack view中后，与外部view的约束被自动去掉了，我们需要设置一下stack view的约束，点击Pin 按钮，弹出菜单设置四个方向的约束，这里的约束默认情况下都是与最近的元素之间的，我们可以通过点击下拉框选择约束的对应关系。
![](/images/2016.01.21/05.png)   

到目前为止，你已经得到了一个扩展到view右侧边缘的stack view，但是现在下面的label还是原来的宽度。你将要学习stack view的alignment属性，能够修复这个问题。   

#### Alignment property  
Alignment属性可以设置stack view元素的对齐方式，对于Axis未Vertical的stack view该属性值可选：Fill、Leading、Center、Trailling，对于Axis为Horizontal的stack view，该属性值可选：Fill、Top、Center、Bottom、FirstBaseline、LastBaseline。
![](/images/2016.01.21/06.png)
![](/images/2016.01.21/07.png)
![](/images/2016.01.21/08.png)
![](/images/2016.01.21/09.png)
选中我们刚才创建的stack view设置Alignment、Distribution都为Fill。我们发现两个label都已经填充到stack view的宽度了。运行一下程序，发现运行良好。

### Convert the "what to see" section
下面转换what to see部分。
1. 选中WHAT TO SEE label和 the <whatToSeeLabel> label。
2. 点击Stack按钮
3. 点击Pin按钮，选中Constrain to margins，设置四个约束：top：20， leading：0， Trailing：0， bottom： 20
4. 设置stack view的Alignment属性为Fill
运行一下程序，发现程序跟原来一样。现在只剩下weather部分了。   

### Convert the weather section
我们可以将weather部分嵌入到一个stack view，但是别忘了Hide按钮，它将会使我们的stack view变得更加复杂。

#### One possible approach
> 这种方案只是探讨，并不会在Xcode中实现。  

你能够将WEATHER label和Hide按钮嵌入到一个横向的stack view中，然后在将新的stack view和<weatherInfoLabel>一起嵌入到一个纵向的stackview中。   

#### Actual approach
我们实际使用的方案是将Hide按钮不嵌入到weather部分的stack view中，设置其与WEATHER label的约束。这里将要添加stack view外部元素与stack view内部元素之间的约束。  

##### Change the weather section – for real
1. 选中WEATHER label和<weatherInfoLabel>
2. 点击Stack按钮。点击Pin按钮，选中Constrain to margins，
3. 设置四个约束：top：20， leading：0， Trailing：0， bottom： 20。
4. 设置stack view的Alignment属性为Fill。设置完如下：
![](/images/2016.01.21/10.png)   

下面我们需要设置Hide按钮与WEATHER的约束。我们需要设置Hide按钮的左边缘与WEATHER右边缘建立约束。但是现在WEATHER充满了整个stack view。我们可以将WEATHER嵌入到一个纵向的stack view中，然后设置alignment属性为Leading。运行我们的程序，发现Hide按钮位置错误了，那是因为我们将WEATHER label嵌入到stack view之后，在Hide按钮上的所有约束都被移除了。我们需要重新为Hide添加约束：从Hide按钮按住Control连线到WEATHER label，在弹出菜单中按住Shift，选中Horizontal Space和Baseline，然后点击Add Constraints：
>我自己做测试的时候，发现不把WEATHER嵌入到纵向的stack view中，直接添加约束也是可以的。

![](/images/2016.01.21/11.png)   

运行程序，Hide按钮的位置现在是正常的了。现在点击Hide按钮，内容被隐藏后，并且空白区域也被下面的内容填充了。现在所有的内容都在一个单独的stack view中，下面将会将其添加到一个统一的stack view中。

### Top-level stack view
1. 选中所有的外层的stack view，嵌入到一个stack view中
![](/images/2016.01.21/12.png)   
2. 点击Pin按钮，选中Constraints to margins，添加四个方向的约束为0
3. 设置spacing为20，Alignment为Fill
storyboard显示如下：
![](/images/2016.01.21/13.png)  
运行程序发现Hide按钮又出问题了。需要重复上面的步骤：从Hide按钮按住Control连线到WEATHER label，在弹出菜单中按住Shift，选中Horizontal Space和Baseline，然后点击Add Constraints。  

运行程序，运行结果变得正常了。

### Repositioning views
将what to see移动到weather上面。选中what to see嵌入的stack view，拖动到weather嵌入的stack view的上面。
![](/images/2016.01.21/14.png)

Hide按钮的位置又错了，选中Hide按钮，点击Update Frames就能够解决了。运行程序，what to see已经移动到weather上面。

### Arranged subviews
UIStackView有一个属性叫做arrangedSubviews，该属性保存着UIStackView中包含的View，其中View的顺序就是View在UIStackView排列的顺序。UIStackView也是UIView的子类，也可以使用addSubview添加子View（例如设置background View），但是在storyboard添加的view都不是subview，都是arrangedSubviews中一个View。  

我们可以使用addArrangedSubview(\_:) or insertArrangedSubview(\_:atIndex:)添加插入View。使用removeArrangedSubview(\_:)从stack view中删除一个View，但是使用该方法删除的View并不会在视图树中真正的删除，需要调用view的removeFromSuperview()才会真正的删除。如果我们直接调用view的removeFromSuperview()，该view在stack view删除的同时还会在视图树中删除。

### Animation
现在点击Hide或者Show按钮的时候，weather详情部分的变化有些生硬，我们可以添加一些动画，让其变得更加好一点。

#### Animating hidden  
UIStackView同样适用于UIView的动画引擎。打开SpotInfoViewController.swift,修改对应方法如下：
{% codeblock lang:swift %}
func updateWeatherInfoViews(hideWeatherInfo shouldHideWeatherInfo: Bool, animated: Bool) {
   let newButtonTitle = shouldHideWeatherInfo ? "Show" : "Hide"
   weatherHideOrShowButton.setTitle(newButtonTitle, forState: .Normal)

   // TODO: Animate when animated == true
   if animated {
       UIView.animateWithDuration(0.3,
           delay: 0.0,
           usingSpringWithDamping: 0.6,
           initialSpringVelocity: 10,
           options: [],
           animations: {
               self.weatherInfoLabel.hidden = shouldHideWeatherInfo
           }, completion: { finished in

           }
       )
   } else {
       weatherInfoLabel.hidden = shouldHideWeatherInfo
   }
 }
 {% endcodeblock %}  

运行程序，运行良好。

#### Animating the axis
下面修改一下rating 部分的动画，使其在不同情况下按照不同的方向显示。
将storyboard的中的rating stack view与对应的类连线，创建IBOutlet属性@IBOutlet weak var ratingStackView: UIStackView!。修改对应方法如下（注释add是新添加的）
 {% codeblock lang:swift %}
 func updateWeatherInfoViews(hideWeatherInfo shouldHideWeatherInfo: Bool, animated: Bool) {
   let newButtonTitle = shouldHideWeatherInfo ? "Show" : "Hide"
   weatherHideOrShowButton.setTitle(newButtonTitle, forState: .Normal)

   // TODO: Animate when animated == true
   if animated {
       UIView.animateWithDuration(0.3,
           delay: 0.0,
           usingSpringWithDamping: 0.6,
           initialSpringVelocity: 10,
           options: [],
           animations: {
               self.weatherInfoLabel.hidden = shouldHideWeatherInfo
           }, completion: { finished in // add
               UIView.animateWithDuration(0.3) {
                   self.ratingStackView.axis = shouldHideWeatherInfo ? .Vertical : .Horizontal
               }
           }
       )
   } else {
       weatherInfoLabel.hidden = shouldHideWeatherInfo
       // add
       self.ratingStackView.axis = shouldHideWeatherInfo ? .Vertical : .Horizontal
   }

 }
 {% endcodeblock %}  
