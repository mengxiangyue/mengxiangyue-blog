title: iOS9 by Tutorials 学习笔记二：App Search
date: 2015-12-27 11:52:58
categories:
  - iOS 9 by Tutoials
tags:
  - iOS 9
---
> 本文为自己读书的一个总结，可能与原书有一定出入  

iOS 9推出了搜索技术，能够让用户在Spotlight中搜索到APP内部的内容。苹果提供了三个APP Search API：
* NSUserActivity
* Core Spotlight
* Web markup

下面简单的说一下我对于这三个API的理解：
1. NSUserActivity:   
NSUserActivity在iOS8就已经提出来了，只是那时候提出来是用作HandOff。在iOS9中它可以用来搜索App中的内容。我们可以把一些想要在Spotlight中被搜到的东西，放到NSUserActivity中，然后就能在Spotlight中被搜到，但是这个有一点限制，就是只能搜索用户访问过得内容。因为UIViewController的userActivity属性继承自UIResponser，只有在UIViewcontroller访问的时候，才有机会设置userActivity属性。   
2. Core Spotlight:   
这个是在iOS9新推出的技术，能够将APP的内容在Spotlight中被搜索到。这个技术我理解：苹果给开发者提供了一个全局的index数据库，我们能够把我们想要能够在Spotlight中搜索的内容，按照苹果的要求放到数据库中，然后苹果就做了其他的事情，让其能够被搜索到。同样我们也可以删除我们存储到数据库中的内容。    
3. Web markup:    
Web Markup在网页上显示App的内容并编入Spotlight索引，如此一来即便没有安装某个App，苹果的索引器也能在网页上搜索特别的标记（markup），在Safari或Spotlight上显示搜索结果。具体会在下一篇文章中详细介绍。

<!--more-->

###  Getting started
下面开始试验一下相关的技术，这里还是利用书中的star工程。现在这个工程运行后，就两个界面：
![](/images/2015.12.27.01.png)    

下面是这个工程的截图：
![](/images/2015.12.27.02.png)    

下面是图中标注的几个关键类的解释：
1. AppDelegate     
  点击搜索结果跳跳转到程序中，会先在这个类里面做一定的处理    
2. EmployeeViewController    
  人员的详细界面，这个里面主要设置NSUserActivity    
3. EmployeeService    
  这个主要是写CoreSpotlight中index相关的东西    
4. EmployeeSearch    
  主要是扩展了Employee类，添加了与搜索相关的属性    
另外工程中有员工相关的一些操作都封装在了一个EmployeeKit的target，由于跟主target不在一个module，所以在主target中需要import。    

在Iphone的Setting/Colleagues/Indexing中有如下三个选项：  
* Disabled 不使用Search API，即不能在Spotlight中搜索到APP中的内容
* ViewedRecords 只有打开过的才能够被搜索到  
* AllRecords 所有的员工信息都能够被搜索到   

#### 搜索我们已经打开过的内容   
使用NSUserActivity实现这个比较简单，只要两个步骤就可以了：
1. 创建NSUserActivity的一个实例，设置相关的属性
2. 赋值给UIViewController的userActivity属性   

下面我们在EmployeeSearch中添加如下代码：  
> 如果没有该文件，需要手动创建一个，然后target选择EmployeeKit   

{% codeblock lang:swift %}
import Foundation
import CoreSpotlight

extension Employee {
  // 这个用于区分Activity，会在点击搜索结果进入APP，相关处理的时候用到，同样也可以在CoreSpotlight中使用到，对于添加、删除index数据的时候都会用到
  public static let domainIdentifier = "com.mengxiangyue.colleagues.employee"
  // 字典 在处理点击的时候，可以根据该字典获取我们想要的数据
  public var userActivityUserInfo: [NSObject: AnyObject] {
    return ["id": objectId]
  }

  // 给Employee添加userActivity属性，主要是方便我们获取userActivity
  public var userActivity: NSUserActivity {
    let activity = NSUserActivity(activityType: Employee.domainIdentifier)
    activity.title = name  // 显示的名字
    activity.userInfo = userActivityUserInfo  // 与该Activity相关的数据
    activity.keywords = [email, department]  // 关键字 表示搜索什么关键字，能够搜索出来该条记录，当然这个只是补充，这里没有添加name，同样也是可以按照name搜索
    return activity
  }
}  
{% endcodeblock %}
这里扩展了Employee，然后添加了几个属性，属性的意义见注释。  
这时候我们需要重新编译一下EmployeeKit（因为与主target不是同一个target）。   

下面打开EmployeeViewController.swift，在viewDidLoad()中添加如下代码：
{% codeblock lang:swift %}
let activity = employee.userActivity
switch Setting.searchIndexingPreference {
case .Disabled:
  activity.eligibleForSearch = false
case .ViewedRecords:
  activity.eligibleForSearch = true
  // relatedUniqueIdentifier 定义一个id 防止NSUserActivity和Core Spotlight重复索引，这里设置为nil，显示一下会重复
  activity.contentAttributeSet?.relatedUniqueIdentifier = nil
case .AllRecords:
  activity.eligibleForSearch = true
}

userActivity = activity
{% endcodeblock %}   

下面在该类中添加如下的方法，用于在合适的时机更新Activity：
{% codeblock lang:swift %}
// 更新NSUserActivity关联的信息
  override func updateUserActivityState(activity: NSUserActivity) {
    activity.addUserInfoEntriesFromDictionary(employee.userActivityUserInfo)
  }
{% endcodeblock %}  

下面在Iphone的Setting/Colleagues/Indexing中选择ViewedRecords。然后启动APP，在列表中点击Brent Reid进入详细页面，然后使用Command+shift+H，计入Home页面，下拉出现搜索框，然后输入brent出现如下界面：  
![](/images/2015.12.27.03.png)   

看到这个搜索结果界面，感觉太难看了，下面我们丰富一下这个搜索结果，苹果提供的搜索结果可以设置如下的内容：  
![](/images/2015.12.27.04.png)   

下面我们在EmployeeSearch.swift添加如下属性：

{% codeblock lang:swift %}
public var attributeSet: CSSearchableItemAttributeSet {
  let attributeSet = CSSearchableItemAttributeSet(itemContentType: kUTTypeContact as String)
  attributeSet.title = name  // 不太清楚是干啥的
  attributeSet.contentDescription = "\(department), \(title)\n\(phone)"
  attributeSet.thumbnailData = UIImageJPEGRepresentation(loadPicture(), 0.9)
  attributeSet.supportsPhoneCall = true
  attributeSet.phoneNumbers = [phone]
  attributeSet.emailAddresses = [email]
  attributeSet.keywords = skills
  attributeSet.relatedUniqueIdentifier = objectId  

  return attributeSet
}
{% endcodeblock %}  

然后将给userActivity添加如下属性：   

{% codeblock lang:swift %}
public var userActivity: NSUserActivity {
  let activity = NSUserActivity(activityType: Employee.domainIdentifier)
  activity.title = name
  activity.userInfo = userActivityUserInfo
  activity.keywords = [email, department]
  activity.contentAttributeSet = attributeSet   // 新添加的这一行
  return activity
}
{% endcodeblock %}  

然后运行程序，搜索结果如下：
![](/images/2015.12.27.05.png)   

但是现在我们注意到，我们点击搜索结果，打开APP并没有按照我们预想的跳转到该员工的详细界面。这个因为我们在程序中没有做对应的处理，下面我们在AppDelete中添加如下的方法：

{% codeblock lang:swift %}
func application(application: UIApplication, continueUserActivity userActivity: NSUserActivity, restorationHandler: ([AnyObject]?) -> Void) -> Bool {
  let objectId: String
  // 先判断了一个type是不是我们自己定义的 然后获取到对应的EmployeeId
  if userActivity.activityType == Employee.domainIdentifier, let activityObjectId = userActivity.userInfo?["id"] as? String {
    objectId = activityObjectId
  }
  // 获取对应Employee实例 然后跳转到对应的界面
  if let nav = window?.rootViewController as? UINavigationController, listVC = nav.viewControllers.first as? EmployeeListViewController, employee = EmployeeService().employeeWithObjectId(objectId) {
    nav.popToRootViewControllerAnimated(false)
    let employeeViewController = listVC.storyboard?.instantiateViewControllerWithIdentifier("EmployeeView") as! EmployeeViewController
    employeeViewController.employee = employee
    nav.pushViewController(employeeViewController, animated: false)
    return true
  }
  return false
}   
{% endcodeblock %}
这时候我们再点击搜索结果就能够跳转到对应的详细界面了。

### CoreSpotlight
下面我们开始使用CoreSpotlight添加这些搜索内容。首先在EmployeeSearch.swift的attributeSet中设置如下属性：

{% codeblock lang:swift %}
// 在前面的代码中已经设置过了
attributeSet.relatedUniqueIdentifier = objectId
{% endcodeblock %}
这个属性主要是将NSUserActivity与Core Spotlight indexed object进行一个关联，防止出现重复的内容（如果出现重复内容，是因为开始的时候测试NSUserActivity的时候没有设置id，还原一下模拟器就好了）   

然后在EmployeeSearch.swift添加如下的代码：

{% codeblock lang:swift %}
// CoreSpotlight需要将一个个item放入其索引数据库中，这里创建一个方便使用
var searchableItem: CSSearchableItem {
  let item = CSSearchableItem(uniqueIdentifier: objectId, domainIdentifier: Employee.domainIdentifier, attributeSet: attributeSet)
  return item
}
{% endcodeblock %}   

然后在EmployeeService.swift添加如下代码：  

{% codeblock lang:swift %}   
import CoreSpotlight

..............<省略一部分代码>

public func indexAllEmployees() {
  let employees = fetchEmployees()
  let searchableItems = employees.map{ $0.searchableItem }
  // 将我们需要被索引的item放入到defaultSearchableIndex中
  CSSearchableIndex.defaultSearchableIndex().indexSearchableItems(searchableItems) { (error) -> Void in
    if let error = error {
      print("Error indexing employees: \(error)")
    } else {
      print("Employees indexed.")
    }
  }
}
{% endcodeblock %}   
然后在设置中选择AllRecords，这时候启动APP，然后搜索，看到的搜索结果如下：
![](/images/2015.12.27.06.png)

但是这时候我们点击搜索结果没有反应，想想应该也能猜到，我们需要在AppDelete中添加代码，最终代码如下：
{% codeblock lang:swift %}
func application(application: UIApplication, continueUserActivity userActivity: NSUserActivity, restorationHandler: ([AnyObject]?) -> Void) -> Bool {
    let objectId: String
    if userActivity.activityType == Employee.domainIdentifier, let activityObjectId = userActivity.userInfo?["id"] as? String {
      objectId = activityObjectId
    }
    // 这部分else是新添加的 使用不一样的type区分NSUserActivity和CoreSpotlight,然后获取对应的objectId，其他的处理都一样了   
    // CSSearchableItemActivityIdentifier这个是CoreSpotlight提供的一个key值
    else if userActivity.activityType == CSSearchableItemActionType, let activityObjectId = userActivity.userInfo?[CSSearchableItemActivityIdentifier] as? String {
      objectId = activityObjectId
    } else {
      return false
    }
    if let nav = window?.rootViewController as? UINavigationController, listVC = nav.viewControllers.first as? EmployeeListViewController, employee = EmployeeService().employeeWithObjectId(objectId) {
      nav.popToRootViewControllerAnimated(false)
      let employeeViewController = listVC.storyboard?.instantiateViewControllerWithIdentifier("EmployeeView") as! EmployeeViewController
      employeeViewController.employee = employee
      nav.pushViewController(employeeViewController, animated: false)
      return true
    }
    return false
  }
{% endcodeblock %}   
这时候我们点击搜索结果应该就能够跳转进入对应的人员详情了。   

### 删除Item
最后在简单的说下删除已经索引的Item，修改EmployeeService.swift对应的方法如下：
{% codeblock lang:swift %}
public func destroyEmployeeIndexing() {
  CSSearchableIndex.defaultSearchableIndex().deleteAllSearchableItemsWithCompletionHandler { (error) -> Void in
    if let error = error {
      print("Error deleting searching employee items: \(error)")
    } else {
      print("Employees indexing deleted.")
    }
  }
}
{% endcodeblock %}
这个方法会在APP启动并且Indxing设置为Disabled的时候调用。   

另外对于CoreSpotlight中对于Item的操作方式还有好多种，这里我就不一一写出来了，有兴趣的可以看看我翻译的API注释，当然文章可能有点老了，但是基本思想应该没变。地址:[CoreSpotlight.framework注释翻译](http://blog.csdn.net/mengxiangyue/article/details/46575977)
