title: iOS9 by Tutorials 学习笔记三：Your App on the Web
date: 2015-12-27 18:22:56
categories:
  - iOS 9 by Tutoials
tags:
  - iOS 9
---

> 这里首先说明一下：这篇文章由于一些限制，我也没有能够进行实验测试，只是尽可能的解释书中的一些知识，可能会有错误，等以后有条件了，我会实验这篇文章中的内容。但是作为了解内容还是不错的。   

在iOS 9之前在iPhone上native和web之间，基本上算是独立的的两部分内容。但是Apple正在努力缩小两者之间的距离，使其越来越近。在iOS 9退出了universal links和web markup，使你能够提供deep links直接进入你的app和在Spotlight和Safari中能够搜索出来你的内容。  

### Getting started   
这一章书中提供了两个工程，一个是APP端的，一个是Server端的，因为这个需要Server端修改一些东西。APP可以通过地址：<https://itunes.apple.com/us/app/rwdevcon-tutorial-conference/id958625272?mt=8>进行下载。APP截图如下：
![](/images/2015.12.27.07.png)   

### Linking to your app
在真正开始前，先回顾一下universal link的前辈：deep links。通过这个回顾，了解一下deep links存在的一些问题。   

#### Deep links
在iOS 9之前我们可以给APP设置URL scheme，在Info.plist里面添加CFBundleURLTypes key。一般格式类似<your app>://。 另外应该也看到过Apple自己的URL scheme，类似tel://、sms://等。   

<!--more-->

一旦设置了URL scheme，就能够通过openURL(\_:)方法调用起来该APP，调用的时候后面可以带着一些参数。然后在我们自己的程序里面可以再AppDelegate的application(\_:handleOpenURL:)中进行对应的处理。这套系统已经存在很久了，但是现在暴露出来一些问题：   
* 安全 UIApplication有一个方法canOpenURL(\_:),可以用来检测用户是否能够打开某个URL secheme，本来苹果的设计是好的，但是不幸的是现在好多开发商使用这个来检测用户手机安排了什么APP，这样就收集了用户的APP列表，涉及到了用户的隐私。   
> canOpenURL(\_:)这个方法在iOS9中有了限制，如果想使用这个方法必须首先把所有的地址添加到info.plist中，不能按照原来由服务器下发来检测APP安装了。  

* 冲突 由于URL scheme是每个APP开发商自己定义的，很有可能两个APP开发商定义相同，这时候如果使用openURL(\_:)，iPhone将不会知道应该怎么处理。   
* No fallback：如果 iOS 试图打开没有注册的 URL scheme，会静默失败，然后用户并不知道发生了什么。   

iOS使用universal links来解决这些问题。使用universal links来代替URL scheme。universal links使用标准的HTTP和HTTPS链接。  

#### Universal links
这里举了一个例子：你有一个域名clownapp.com，你可以注册http://clownapp.com作为你的universal link。如果用户安装了你的clownapp。当他在Safari或者web view中点击链接http://clownapp.com/clowns/fizbo的时候，将会直接进入到你的APP的fizbo的profile页面。如果你没有安装这个将会直接跳转到你的网站上的fizbo的profile页面。如果你使用openURL(\_:)打开，也会与这个动作一样。   
> PS: 这里我运行书中的例子，在模拟器的Safari中打不开。可能是我的原因   

Universal links与deep links有如下的有点：  
* 唯一 由于使用的是域名，能够保证唯一性
* 安全 将你的app与你的域名绑定，上传一个安全签名到你的网站服务器。同样其他的APP也不会轻易的知道手机上是否安装了你的APP。  
> 这里原文如下There's also no way for other apps to tell whether your app is installed.这里不是没有方式，只是说没有原来那么容易。使用URL scheme白名单的方式还是能够检测。

* 简单 由于跳转到APP和服务器的链接统一了，所以不用考虑在APP和手机上需要使用两套不同的链接了。

#### 注册你的App，使其能够处理universal links   
为了使App能够处理对应的链接，首先需要让App知道应该处理什么链接。这里使用的链接是rwdecon.com。按照下图添加对应的链接：  
![](/images/2015.12.27.08.png)  

> 这里可能会出现选择账户，这时候就选择你对应的就好了，如果没有账户可以进入到Account添加。   

#### 注册你的服务器能够处理unilateral links  
你需要在服务器的根目录下面，添加文件名为apple-app-site-association(没有后缀)的一个文件，然后在里面添加上如下的内容：
{% codeblock lang:swift %}
{
    "applinks": {
        "apps": [],
        "details": [
            {
                "appID": "KFCNEC27GU.com.razeware.RWDevCon",
                "paths": [
                    "/videos/\*"
                ]
            }
        ]
    }
}
{% endcodeblock %}  
其中的appId是由team ID和bundle ID拼成的。Paths 数组包含了一个你的App应该处理的 URLs 白名单，这个 paths 数组还支持 基本的模式匹配，例如 \*，？ 等，如 /videos/\*/year/201?/videoName。   

这个文件需要上传到服务器的根目录，并且能够通过HTTPS访问到，并且没有重定向。

#### 在你的App上处理universal links
> 这部分代码没有试验    

上面已经添加对应的universal links，下面需要在App中处理对应的链接了。这里需要解析对应的链接，然后做一些相关的业务逻辑。在Session.swift添加下面的方法，这个方法主要是用来解析对应的url的：
{% codeblock lang:swift %}
class func sessionByWebPath(path: String,
context: NSManagedObjectContext) -> Session? {

  let fetch = NSFetchRequest(entityName: "Session")
  fetch.predicate = NSPredicate(format: "webPath = %@", [path])

  do {
    let results = try context.executeFetchRequest(fetch)
    return results.first as? Session
  } catch let fetchError as NSError {
    print("fetch error: \(fetchError.localizedDescription)")
  }

  return nil
}
{% endcodeblock %}  

在AppDelegate.swift添加如下方法：
{% codeblock lang:swift %}
extension AppDelegate {
  // 辅助方法
  func presentVideoViewController(URL: NSURL) {
    let storyboard = UIStoryboard(name: "Main", bundle: nil)
    let navID = "NavPlayerViewController"

    let navVideoPlayerVC =
    storyboard.instantiateViewControllerWithIdentifier(navID)
      as! UINavigationController

    navVideoPlayerVC.modalPresentationStyle = .FormSheet

    if let videoPlayerVC = navVideoPlayerVC.topViewController
      as? AVPlayerViewController {

        videoPlayerVC.player = AVPlayer(URL: URL)

        let rootViewController = window?.rootViewController
        rootViewController?.presentViewController(navVideoPlayerVC,
          animated: true, completion: nil)
    }
  }

  func application(application: UIApplication,
    continueUserActivity
    userActivity: NSUserActivity,
    restorationHandler: ([AnyObject]?) -> Void) -> Bool {

      //1 系统用 NSUserActivityTypeBrowsingWeb 表示对应的 universal HTTP links
      if userActivity.activityType ==
        NSUserActivityTypeBrowsingWeb {

          let universalURL = userActivity.webpageURL!

          //2 提取出 url 的不同部分
          if let components = NSURLComponents(URL: universalURL,
            resolvingAgainstBaseURL: true),
            let path = components.path {

              if let session = Session.sessionByWebPath(path,
                context: coreDataStack.context) {
                  //3 找到 session，然后播放 video
                  let videoURL = NSURL(string: session.videoUrl)!
                  presentVideoViewController(videoURL)
                  return true
              } else {
                //4 无法理解就打开网站首页
                let app = UIApplication.sharedApplication()
                let url = NSURL(string: "http://www.rwdevcon.com")!
                app.openURL(url)
              }
          }
      }
      return false
  }
}
{% endcodeblock %}   

下面有两个链接，可以给自己写一封邮件带上下面的两个链接，第一个是能够正常打开视频播放的，第二个直接打开网站首页。PS：我没有试验成功
{% codeblock lang:swift %}  
good link
http://www.rwdevcon.com/videos/talk-tammy-coron-possible.html
bad link
http://www.rwdevcon.com/videos/tim-cook-keynote.html
{% endcodeblock %}   

### 使用web markup
Search 包含三种不同的 API：NSUserActivity，CoreSpotlight，web markup。前两种已经介绍过了，现在来看第三种。

你可以使用 web markup 在搜索结果中得到你 app 应用里面的内容。如果你有一个网站，内容与 APP 的内容一致，你可以使用基本的 markup、Smart App Banners、native App能够处理universal links来修改你的网站，使其能够更好的被搜索、展示。   

苹果有自己的爬虫，如果你的网站使用web markup，苹果的爬虫能够收集到对应的信息，然后保存到自己的服务器上，然后其他用户在搜索的时候能够搜索到对应的内容，不管用户是否安装了你的App，这样也能够帮助你获取一部分用户。   

#### 使你的网站能够被发现   
苹果的爬虫会到处去爬数据，但是不一定能够很快的发现你的网站，这里有个方法能够帮助苹果爬虫发现你的网站。
1. 在iTunes Connect中，在设置**Support URL**的地方，设置**Marketing URL**，指向你已经使用markup的网站。  
![](/images/2015.12.27.09.png)
2. 保证你填写的URL能够被苹果的爬虫访问到。
3. 检查你Robots.txt文件，保证苹果的爬虫能够正常的爬取你的网站。PS:关于Robots.txt自行百度吧。

#### 添加Smart App Banners
添加了Smart App Banners后，打开网站的时候会在顶部出现一个banner,对于已经安装App的用户，会显示一个OPEN按钮方便用户打开对应的App，对于未安装App的用户，将会出现一个view按钮，点击将会进入App store下载该App。效果图类似如下：
![](/images/2015.12.27.10.png)   

实现这个效果的方式，在你想要添加banner的网页上添加如下代码：
{% codeblock lang:html %}
<meta name="apple-itunes-app" content="app-id=958625272, app- argument=http://www.rwdevcon.com/videos/talk-ray-wenderlich-teamwork.html">
{% endcodeblock %}
这里的name是App在store中的名字，下面的content包含两部分内容：
* app-id 在store上的app id
* app- argument 包含跳转回 App 的 URL，iOS 9 之前这个参数是自定义的 URL scheme deep link，现在 Apple 推荐使用 HTTP/HTTPS universal links
> Smart App Banners 仅仅支持 Safari   

你能使用Applebot支持的开放的mobile links，比如：Twitter Cards和App Links，但是这两种标记我自己也没有试验，所以只是贴出来代码：
{% codeblock lang:html %}
// Twitter Cards  具体 https://dev.twitter.com/cards/mobile
<meta name="twitter:app:name:iphone" content="RWDevCon">
<meta name="twitter:app:id:iphone" content="958625272">
<meta name="twitter:app:url:iphone" content="http://www.rwdevcon.com/ videos/talk-ray-wenderlich-teamwork.html">

// App Links 具体http://applinks.org
<meta name="twitter:app:name:iphone" content="RWDevCon">
<meta name="twitter:app:id:iphone" content="958625272">
<meta name="twitter:app:url:iphone" content="http://www.rwdevcon.com/ videos/talk-ray-wenderlich-teamwork.html">
{% endcodeblock %}    

#### Semantic markup using Open Graph  
苹果爬虫爬到你的内容并不保证会显示在 Spotlight 的搜索结果中，因为他还会和其他搜索结果内容进行竞争。   

Apple 并没有公布具体的评级算法，只是确保你的内容会被考虑。而当用户明显地点击或搜索结果与你的内容高度相关，那么就会优先被 Apple 考虑。

最后，Apple 建议为 markup 添加一些结构化的数据，来使其更好地以富文本的形式显示在 Spotlight 中。
{% codeblock lang:html %}
<meta property="og:image" content="http://www.rwdevcon.com/assets/images/  
videos/talk-ray-wenderlich-teamwork.jpg" />  
<meta property="og:image:secure_url" content="https://www.rwdevcon.com/  
assets/images/videos/talk-ray-wenderlich-teamwork.jpg" />  
<meta property="og:image:type" content="image/jpeg" />  
<meta property="og:image:width" content="640" />  
<meta property="og:image:height" content="340" />  
<meta property="og:video" content="http://www.rwdevcon.com/videos/Ray-  
Wenderlich-Teamwork.mp4" />  
<meta property="og:video:secure_url" content="https://www.rwdevcon.com/  
videos/Ray-Wenderlich-Teamwork.mp4" />  
<meta property="og:video:type" content="video/mp4" />  
<meta property="og:video:width" content="1280" />  
<meta property="og:video:height" content="720" />  
<meta property="og:description" content="Learn how teamwork lets you  
dream bigger, through the story of an indie iPhone developer who almost  
missed out on the greatest opportunity of his life." />
{% endcodeblock %}

> 上面的og后面的属性，我也没有找到出处，有谁清楚麻烦留言说明一下。谢谢

关于web markup相关的详细的东西可以看苹果的文档<https://developer.apple.com/library/ios/documentation/General/Conceptual/AppSearch/WebContent.html>

最后说明一下：这篇文章由于一些资源问题，我没有做什么测试，可能有地方不对，如果哪里错误了，请指出来，谢谢。

突然感觉这是最没底的一篇文章。
