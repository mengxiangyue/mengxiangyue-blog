title: iOS9-by-Tutorials-学习笔记九：3D Touch
date: 2016-02-23 22:40:27
categories:
  - iOS 9 by Tutoials
tags:
  - iOS 9
---

很久没有更新文章了，自己也感觉自己懒得要发霉了。为了让自己不至于发霉，今天开始继续更新文章。   

本篇文章介绍一下3D Touch。3D Touch 是iPhone 6s推出的一项新的技术，利用该技术能够更快的预览内容，更加平滑的进行多任务等。Apple提供一下与3D Touch 相关的API：   
* UITouch有一个 **force** 属性用来检测用户按下的程度。
* UIViewController提供了一个新的API，允许你展示新的view Controller的预览，这个预览成为 **peek**。此时继续用力，将会 **pop** 该预览，进入对应的View Controller 界面。
* UIApplicationShortcutItem 是一个新的类，你能使用它给home屏幕上的icon添加快捷操作。

### Getting Started
在这篇文章中将会使用一个叫做 **Doodles(涂鸦)** App, 这个APP 比较简单，就两个界面：列表+详情，截图如下：
![](/images/2016.02.23/01.png)  

<!--- more --->  

代码结构如下：
![](/images/2016.02.23/02.png)   

### UITouch force
iOS9后，UITouch 添加了一个force属性，这个属性表示的按压屏幕的力量，取值范围为0~maximumPossibleForce(touch.maximumPossibleForce,这个值不是1)。我们可以利用force属性去改造我们的APP，让线条的粗细随着按压屏幕力量的变化而变化。
修改Canvas.swift中方法如下（下面标记update的为修改的）：
{% codeblock lang:swift %}
private func addLineFromPoint(from: CGPoint, toPoint: CGPoint, withForce force: CGFloat = 1) { // update
    UIGraphicsBeginImageContextWithOptions(bounds.size, false, 0.0)
    print("force:\(force)")
    drawing?.drawInRect(bounds)

    let cxt = UIGraphicsGetCurrentContext()
    CGContextMoveToPoint(cxt, from.x, from.y)
    CGContextAddLineToPoint(cxt, toPoint.x, toPoint.y)

    CGContextSetLineCap(cxt, .Round)

    CGContextSetLineWidth(cxt, strokeWidth * force) // update

    strokeColor.setStroke()

    CGContextStrokePath(cxt)

    drawing = UIGraphicsGetImageFromCurrentImageContext()

    layer.contents = drawing?.CGImage

    UIGraphicsEndImageContext()
  }
{% endcodeblock %}  

继续修改如下方法：
{% codeblock lang:swift %}
public override func touchesMoved(touches: Set<UITouch>, withEvent event: UIEvent?) {
    if let touch = touches.first {
        // 先去验证是否支持3D Touch
        if traitCollection.forceTouchCapability == .Available {
            addLineFromPoint(touch.previousLocationInView(self), toPoint: touch.locationInView(self), withForce: touch.force)
        } else {
            addLineFromPoint(touch.previousLocationInView(self), toPoint: touch.locationInView(self))
        }
    }
  }
{% endcodeblock %}  

修改完后运行一下，用不同压力绘画，效果如下：
![](/images/2016.02.23/03.png)  

### Peeking and popping
对于 **Peek** 和 **Pop** 等写完下面的例子自然就明白了。  

修改DoodlesViewController.swift对应方法如下：
{% codeblock lang:swift %}
override func viewDidLoad() {
  super.viewDidLoad()
  // update
  if traitCollection.forceTouchCapability == .Available {
      // 添加对于3D Touch的代理
      registerForPreviewingWithDelegate(self, sourceView: view)
  }
}

…………

// 扩展该类实现对应的协议 有两个方法用于处理peek pop
//MARK: - 3D Touch
extension DoodlesViewController: UIViewControllerPreviewingDelegate {
    // peek 回调 返回一个包含对应内容的UIViewController
    func previewingContext(previewingContext: UIViewControllerPreviewing, viewControllerForLocation location: CGPoint) -> UIViewController? {
        guard let indexPath = tableView.indexPathForRowAtPoint(location), cell = tableView.cellForRowAtIndexPath(indexPath) as? DoodleCell else { // 获取点击的cell
            return nil
        }
        let identifier = "DoodleDetailViewController"
        guard let detailVC = storyboard?.instantiateViewControllerWithIdentifier(identifier) as? DoodleDetailViewController else { // 创建对应的View Controller
            return nil
        }
        detailVC.doodle = cell.doodle // 设置数据

        // 显示那部分区域响应的3D Touch 在peek之前，该frame之外的内容将会被模糊
        previewingContext.sourceRect = cell.frame //CGRect(x: 0, y: 100, width: 300, height: 40)// cell.frame
        return detailVC
    }

    // pop commitViewController是上面方法返回的
    func previewingContext(previewingContext: UIViewControllerPreviewing, commitViewController viewControllerToCommit: UIViewController) {
        // peek之后继续用力 将会调用该方法 进入对应的view Controller
        showViewController(viewControllerToCommit, sender: self)
    }
}
{% endcodeblock %}   

运行效果如下：
![](/images/2016.02.23/04.png)  

#### Preview actions
在 **Peek** 状态下，我们可以添加一些快捷操作，通过上划就能显示出来这些快捷操作。这些快捷操作是需要添加到 **Peek** 显示的View Controller中的。
在DoodleDetailViewController.swift 末尾添加如下代码：
{% codeblock lang:swift %}
weak var doodlesViewController:  DoodlesViewController?
override func previewActionItems() -> [UIPreviewActionItem] {
    // UIPreviewAction 专门为3D Touch使用
    let shareAction = UIPreviewAction(title: "分享", style: .Default) { (previewAction, viewController) -> Void in
        if let doodlesVC = self.doodlesViewController, activityViewController = self.activityViewController {
            doodlesVC.presentViewController(activityViewController, animated: true, completion: nil)
        }
    }

    let deleteAction = UIPreviewAction(title: "删除", style: .Default) { (previewAction, viewController) -> Void in
        guard let doodle = self.doodle else {
            return
        }
        Doodle.deleteDoodle(doodle)
        if let doodlesViewController = self.doodlesViewController {
            doodlesViewController.tableView.reloadData()
        }
    }
    return [shareAction, deleteAction]
}
{% endcodeblock %}
上面的代码添加了两个快捷键，一个是分享一个是删除。

在DoodlesViewController.swift中也需要修改一下代码：
{% codeblock lang:swift %}
// 注释new add 是新添加的
func previewingContext(previewingContext: UIViewControllerPreviewing, viewControllerForLocation location: CGPoint) -> UIViewController? {
    guard let indexPath = tableView.indexPathForRowAtPoint(location), cell = tableView.cellForRowAtIndexPath(indexPath) as? DoodleCell else { // 获取点击的cell
        return nil
    }
    let identifier = "DoodleDetailViewController"
    guard let detailVC = storyboard?.instantiateViewControllerWithIdentifier(identifier) as? DoodleDetailViewController else { // 创建对应的View Controller
        return nil
    }
    detailVC.doodle = cell.doodle // 设置数据

    detailVC.doodlesViewController = self // new add

    // 显示那部分区域响应的3D Touch 在peek之前，该frame之外的内容将会被模糊
    previewingContext.sourceRect = cell.frame //CGRect(x: 0, y: 100, width: 300, height: 40)// cell.frame
    return detailVC
}
{% endcodeblock %}    

运行程序，在 **Peek** 状态上划效果如下：
![](/images/2016.02.23/05.png)   

### Home screen quick actions  
3D Touch 另一个作用就是能够在Home界面的图标上，通过按压在图标的周围显示一个快捷操作（shortcuts）的菜单，通过这个菜单，我们能够更加快速直接的进入某项功能。现在主流的App都已经支持这项功能，可以在图标上按按试试。   

这种快捷操作（shortcuts）分两种：
* **Static shortcuts** 静态的，在Info.plist里面配置，在程序安装后就会存在。
* **Dynamic shortcuts** 动态的，在运行时创建的，直到程序第一次运行后才会存在。

#### Adding a static shortcut  
添加一个静态的shortcut，首先需要在Info.plist里面配置，按照下图进行相应的配置：
![](/images/2016.02.23/06.png)

配置好了之后运行程序，在Home页面App Icon上按压能够看到我们配置的shortcut，点击后发现应用启动，但是不是我们想要的结果。我们需要在Appdelegate.swift中添加如下的代码：
{% codeblock lang:swift %}   
// 响应shortcut的点击
func application(application: UIApplication, performActionForShortcutItem shortcutItem: UIApplicationShortcutItem, completionHandler: (Bool) -> Void) {
    handleShortcutItem(shortcutItem)

    // 如果你自己处理了shortcut 这里参数为true 否则为false
    completionHandler(true)
}

func handleShortcutItem(shortcutItem: UIApplicationShortcutItem) {
    print(shortcutItem.userInfo)
    switch shortcutItem.type {
    case "com.razeware.Doodles.new":
        presentNewDoodleViewController()
    default:
        break
    }
}

func presentNewDoodleViewController() {
    let identifier = "NewDoodleNavigationController"
    let doodleViewController = UIStoryboard.mainStoryboard .instantiateViewControllerWithIdentifier(identifier)

    window?.rootViewController? .presentViewController(doodleViewController, animated: true, completion: nil)
}
{% endcodeblock %}  

运行程序，点击shortcut会进入新建View Controller。   

#### Adding a dynamic shortcut
当然我们也能动态的通过程序添加shortcut，这个有一个限制就是程序必须运行一次才会生效。   
修改Doodle.swift对应的方法如下（注释add地方为新加的）：
{% codeblock lang:swift %}
static func addDoodle(doodle: Doodle) {
  allDoodles.append(doodle)
  Doodle.configureDynamicShortcuts() // add
}

static func deleteDoodle(doodle: Doodle) {
  if let index = allDoodles.indexOf({ $0.name == doodle.name }) {
    allDoodles.removeAtIndex(index)
      Doodle.configureDynamicShortcuts() // add
  }
}

// 这个方法添加
static func configureDynamicShortcuts() {
    if let mostRecentDoodle = Doodle.sortedDoodles.first {
        let shortcutType = "com.razeware.Doodles.share"
        let shortcutItem = UIApplicationShortcutItem(type: shortcutType, localizedTitle: "Share latest Doodle", localizedSubtitle: mostRecentDoodle.name, icon: UIApplicationShortcutIcon(type: .Share), userInfo: nil)
        UIApplication.sharedApplication().shortcutItems = [shortcutItem]
    } else {
        UIApplication.sharedApplication().shortcutItems = []
    }
}
{% endcodeblock %}   

修改AppDelegate.swift对应的方法：
{% codeblock lang:swift %}
func application(application: UIApplication, didFinishLaunchingWithOptions launchOptions: [NSObject: AnyObject]?) -> Bool {
  configureAppAppearance()
  Doodle.configureDynamicShortcuts() // add
  return true
}

…………

func handleShortcutItem(shortcutItem: UIApplicationShortcutItem) {
    print(shortcutItem.userInfo)
    switch shortcutItem.type {
    case "com.razeware.Doodles.new":
        presentNewDoodleViewController()
    case "com.razeware.Doodles.share": // 处理share
        shareMostRecentDoodle()
    default:
        break
    }
}

func shareMostRecentDoodle() {
    guard let mostRecentDoodle = Doodle.sortedDoodles.first, navigationController = window?.rootViewController as? UINavigationController else {
        return
    }

    let identifier = "DoodleDetailViewController"
    let doodleViewController = UIStoryboard.mainStoryboard.instantiateViewControllerWithIdentifier(identifier) as! DoodleDetailViewController

    doodleViewController.doodle = mostRecentDoodle
    doodleViewController.shareDoodle = true
    navigationController.pushViewController(doodleViewController, animated: true)
}

{% endcodeblock %}   
运行程序，然后退出到Home界面，按压图标，点击share，能够进入程序对应的详情页面，调用起来分享。
![](/images/2016.02.23/07.png)  

3D Touch相关的内容到这里就已经更新完了，初步计划以后录制视频关于动画相关的，希望有兴趣的朋友关注一下。
