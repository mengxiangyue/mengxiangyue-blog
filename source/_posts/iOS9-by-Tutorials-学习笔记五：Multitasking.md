title: iOS9 by Tutorials 学习笔记五：Multitasking
date: 2016-01-07 23:10:56
categories:
  - iOS 9 by Tutoials
tags:
  - iOS 9
---

在WWDC 2015上，苹果推出了multitasking，这个功能允许我们同时运行两个App，比如在看视频的时候，能够同时刷微博，由于是同时运行两个App，对于硬件的要求比较高，目前苹果并没有在所有的设备上面开放这些功能。下面就简单的介绍一下multitasking。

### Getting started
在书中与本章配套的有一个工程Travelog，打开这个工程，在iPad Air 2模拟器上运行一下。我们切换横竖屏，你能看到类似如下的界面：
![](/images/2016.01.09/01.png)

<!---more--->
Travelog App使用了UISpiltViewController来展示左边的列表，点击左边的列表，在右边会显示对应的详细信息。我们会更具这个App介绍一下multitasking的三种模式：
* Slide Over
* Split View
* Picture in Picture, or PIP


#### Slide Over
我们沿着iPad的屏幕右边缘像左边滑动，这时候会浮动出来一个App的列表，点击其中的一个App，会显示如下，这种状态就是Slide Over模式：
![](/images/2016.01.09/02.png)

在这种状态下，Travelog是不能使用的，只有我们弹出的日历App是可以使用的。


#### Split View
注意一下在日历App左边有一个小的图标，点击那个图标后，Travelog将会和日历App同时显示在iPad的屏幕上，并且同时能够使用，这种状态就是Split View模式：
![](/images/2016.01.09/03.png)    


#### Picture in Picture, or PIP
这种模式翻译成成中文是画中画，类似于电视上的画中画的功能。一个例子是我们可以在使用FaceTime的时候，同时使用其他的App。FaceTime将会被缩放到一个小的界面中，并且浮动在你使用的App上面。

#### multitasking支持情况
![](/images/2016.01.09/04.png)   

### 让你的App支持multitasking
如果你的App慢如如下的条件，那么你的App就能够支持multitasking。
* 是一个universal app
* 使用SDK 9.x编译
* 支持所有的方向
* 使用launch storyboard   

满足上面四个条件的APP只是能够支持multitasking，并不表示能够完美的适配。   
Travelog 满足了上面的四个功能，但是它并没有完美的适配。   

#### Orientation and size changes   
在Split View 模式下面运行 Travelog，旋转iPad为竖屏方向，显示如下：
![](/images/2016.01.09/05.png)   

这是一个列表界面，能够看到左边有很大的空白，在后面的内容中，我们能够更好的利用这些空白区域。   

旋转屏幕至横屏状态，如下：
![](/images/2016.01.09/06.png)  

上面的界面也是列表界面，但是master列太窄了。   

打开SplitViewController.swift文件，SplitViewController是UISplitViewController的子类，并且覆写了viewDidlayoutSubviews()， 用来更新主列的最大宽度。这个方法可能不起作用，因为在横屏Split view 模式下也可能出现窄的windown。  

UIKit提供了下面几个方法用于捕获你的layout的变化:   
1. willTransitionToTraitCollection(\_:, withTransitionCoordinator:)
2. viewWillTransitionToSize(\_:, withTransitionCoordinator:)
3. traitCollectionDidChange(\_:):   

下面展示了Size Classes的各种状态：
![](/images/2016.01.09/07.png)

>上图中 R 代表 Regular 而 C 代表 Compact   

根据上图可以看出，并不是所有的multitasking和方向改变都会触发size class改变，所有你不能仅仅依靠size classes去提供最好的用户体验。我们可以viewWillTransitionToSize这个方法，在size classes改变的时候，做出正确的操作。   

下面实际操作一下，在SplitViewController.swift删除viewDidLayoutSubviews() 和 updateMaximumPrimaryColumnWidth()方法，添加某些方法，最终如下（看注释）：
{% codeblock lang:swift %}
import UIKit

class SplitViewController: UISplitViewController {

//  需要删除
//  override func viewDidLayoutSubviews() {
//    super.viewDidLayoutSubviews()
//    updateMaximumPrimaryColumnWidth()
//  }

  override func preferredStatusBarStyle() -> UIStatusBarStyle {
    return .LightContent
  }

//  需要删除
//  // MARK: Helper
//  
//  func updateMaximumPrimaryColumnWidth() {
//    if UIInterfaceOrientationIsPortrait(UIApplication.sharedApplication().statusBarOrientation) {
//      maximumPrimaryColumnWidth = 170.0
//    } else {
//      maximumPrimaryColumnWidth = UISplitViewControllerAutomaticDimension
//    }
//  }

    // 添加的
    override func viewDidLoad() {
        super.viewDidLoad()
        updateMaximumPrimaryColumnWidthBasedOnSize(view.bounds.size)
    }

    // 添加的
    // 这是一个辅助方法 用于设置主列的最大宽度
    func updateMaximumPrimaryColumnWidthBasedOnSize(size: CGSize) {
        if size.width < UIScreen.mainScreen().bounds.width || size.width < size.height {
            maximumPrimaryColumnWidth = 170.0
        } else {
            maximumPrimaryColumnWidth = UISplitViewControllerAutomaticDimension
        }
    }

    // 添加的
    override func viewWillTransitionToSize(size: CGSize, withTransitionCoordinator coordinator: UIViewControllerTransitionCoordinator) {
        super.viewWillTransitionToSize(size,withTransitionCoordinator: coordinator)
        updateMaximumPrimaryColumnWidthBasedOnSize(size)

    }

}
{% endcodeblock %}   

运行程序app最终效果如下，能够看出来app的主列的宽度体验更好一点了。
![](/images/2016.01.09/08.png)    

仔细看一下没行的内容，能够看到我们行上的内容，还是有问题的。应该是我们的cell没有响应size的变化。打开LogCell.swift找到layoutSubviews()的实现，能够发现代码检查的是UIScreen.mainScreen().bounds.width，再决定cell是使用Compact view还是regular view。 UIScreen代表整个屏幕，不去理会multitasking的状态。你不能再依赖屏幕尺寸来判断了。更新如下代码：
{% codeblock lang:swift %}
// 修改
  static let widthThreshold: CGFloat = 180.0
......
override func layoutSubviews() {
   super.layoutSubviews()
   // 修改
//    let isTooNarrow = UIScreen.mainScreen().bounds.width < LogCell.widthThreshold
   let isTooNarrow = bounds.width <= LogCell.widthThreshold
   compactView.hidden = !isTooNarrow
   regularView.hidden = isTooNarrow
 }
{% endcodeblock %}

修改后运行效果如下：
![](/images/2016.01.09/09.png)

### Adaptive presentation
在如下状态下面点击 Photo Library bar button，能够看到如下效果：
![](/images/2016.01.09/10.png)

在上图状态下拖动中间的标志，使APP为5：5模式，效果如下：
![](/images/2016.01.09/11.png)   

能够注意到，我们没有做任何改变，但是我们从33%变成50%的时候，弹出的模态菜单变成了整个页面。这个效果是由UIPopoverPresentationController控制的，而我们想要的效果是是只有APP在Slide Over模式或者作为第二APP并且在33%的时候才会使用popover。我们可以设置UIPopoverPresentationController的delegate来实现我们的行为。   

打开LogsViewController.swift，添加如下代码：
{% codeblock lang:swift %}
extension LogsViewController: UIPopoverPresentationControllerDelegate {

    func adaptivePresentationStyleForPresentationController( controller: UIPresentationController,traitCollection: UITraitCollection) -> UIModalPresentationStyle {

        //1 判断是iPad
        guard traitCollection.userInterfaceIdiom == .Pad else {
            return .FullScreen
        }

        // 宽度大于320使用popover
        if splitViewController?.view.bounds.width > 320 {
            return .None
        } else {
            return .FullScreen
        }
    }

}
{% endcodeblock %}  

下面找到如下方法设置代理：
{% codeblock lang:swift %}
func presentImagePickerControllerWithSourceType(sourceType: UIImagePickerControllerSourceType) {
    let controller = UIImagePickerController()
    controller.delegate = self
    controller.sourceType = sourceType
    controller.mediaTypes = [String(kUTTypeImage), String(kUTTypeMovie)]
    controller.view.tintColor = UIColor.themeTineColor()
    if sourceType == UIImagePickerControllerSourceType.PhotoLibrary {
      controller.modalPresentationStyle = .Popover
      let presenter = controller.popoverPresentationController
      presenter?.sourceView = view
      presenter?.barButtonItem = photoLibraryButton
      presenter?.delegate = self // 添加
    }
    presentViewController(controller, animated: true, completion: nil)
  }
{% endcodeblock %}

最终运行的效果如下：
![](/images/2016.01.09/12.png)

算是完成了这一篇笔记吧，可能跟书中不完全一样，感觉这篇文章只是简单介绍了一下，如果想更多了解请查看：[苹果文档](https://developer.apple.com/library/ios/documentation/WindowsViews/Conceptual/AdoptingMultitaskingOniPad/index.html#//apple_ref/doc/uid/TP40015145-CH3-SW1)
