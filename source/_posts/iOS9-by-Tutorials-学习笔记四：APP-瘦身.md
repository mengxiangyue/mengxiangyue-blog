title: iOS9 by Tutorials 学习笔记四：APP 瘦身
date: 2016-01-04 22:26:09
categories:
  - iOS 9 by Tutoials
tags:
  - iOS 9
---

>这篇文章在书中的标题是App Thinning，这里我给翻译成了App 瘦身。

>本文然然进行了一些语法的修改，很开心她为我修改这些东西。她说我转折只会用但是，被她这么一说想想还真是只是会用但是，嘿嘿。  

iPhone经过这几年的发展，已经发生了很大的变化，例如屏幕变得更加多样，尺寸更多，内存变得更大，CPU的架构也在变化。伴随着iPhone的变化，iOS也在变化，例如AutoLayout、size classes、split view controller等。这些技术及设备的变化给我在开发的过程中也造成了许多的问题，不仅如此苹果通过不断推出新的技术，努力在帮助我们使用同一套代码开发适应多个设备的Universal的App。另一方面Universal App虽然在开发的过程中，方便了我们开发人员，可是对于用户来说就不那么友好了，由于需要适配多种设备，所以里包含所有设备的代码，但真正的在运行的时候，我们并不需要那么多相关的代码及资源。    

例如下面的一张图，是一个App运行在iPhone 6+上，使用的各个资源相关的情况：
![](/images/2016.01.04.01.png)   

上图中对勾标出来的是在iPhone 6+上真实运行的时候使用到的相关的资源及代码，对比有对勾的部分，更多的是没有被对勾标出来的部分。可以想象我们下载了一个App（前提这个App是Universal的），然后至少一半的代码及资源是我们不需要的，白白占用着我们的空间。这样对用户体验也不好。为了解决这个问题苹果在iOS 9给出了新的解决方案：

<!---more--->

* App Slicing 当你提交你的iOS 9 打包文件到App Store的时候，苹果编译你的资源和可执行文件，然后为每个设备生成一个特定的可执行文件。最终，设备只会下载适应与其特性的，并且它使用到的内容。这些特性包含显卡性能（原文单词：graphics capabilities）、内存级别、CPU架构、size classes、屏幕 scaling等。  
* On Demand Resouces 应用程序的资源只有在需要使用的时候才会下载，并且如果其他资源需要空间这些资源可以被移除。
* Bitcode 在你提交App到App Store的时候，Bitcode可以作为中间产物一起提交。包含bitcode配置的程序将会在App store上被编译和链接。bitcode允许苹果在后期重新优化我们程序的二进制文件，而不需要我们重新提交一个新的版本到App store上。   

这三个技术加起来，统一称为App Thinning。

### Getting started   
打开本章节的初始项目，然后选在iPad Air 2运行，这时候运行效果如下：
![](/images/2016.01.04.02.png)

伴随着模拟器启动起来的还打开了一个Finder窗口：  
![](/images/2016.01.04.03.png)

这个Finder窗口能够打开，是因为在程序中添加了一个脚本，每次运行的时候都会执行，脚本所在地方如下：
![](/images/2016.01.04.04.png)
{% codeblock lang:bin %}
echo "App Size in KB:  `du -sk \"${CONFIGURATION_BUILD_DIR}/${EXECUTABLE_NAME}.app\"`"
if [ "${CONFIGURATION}" = "Debug" ]; then
open ${CONFIGURATION_BUILD_DIR}
fi
{% endcodeblock %}   

在Finder的Old CA Maps点击右键，选择显示包内容，如下：
![](/images/2016.01.04.05.png)   

上图中标注的说明如下：
1. Assets.car是Assets.xcassets被Xcode进行编译后的文件。
2. Old CA Maps是真实运行在设备上的可执行文件。
3. Santa Cruz PNGs 这个是图片文件，但是没有被编译到Assets.car文件中，这是因为它并没有放到Assets.xcassets中，而是放到了工程的顶层文件中。
4. SD_Map.bundle 这个就是地图图片文件，但是将近120MB。

#### Measuring your work   
本章介绍一些App瘦身相关的东西，所以我们必须能够测量App是否减少了。工程里面已经内置了一个脚本（上面代码里面有），能够在build的过程中输出App的大小。查看的位置如下：
![](/images/2016.01.04.06.png)   

### Slicing up app slicing
App slicing包含两部分内容：可执行文件分片（Executable slicing）和资源分片（resource slicing）。    

**Executable slicing** 指的是在设备下载App的时候会根据设备的相关信息只是下载对应该设备的相关的可执行文件，并不会包含其他设备及架构的可执行文件，达到App安装包的缩小。并且这个功能并不需要我们做太多，App Store默认支持的。   

默认情况下提交到App Store的包是包含所有的内容的，这些都在配置文件里面，App Store会自动创建对应于每个类型的可执行文件。这个在iOS9+上支持。

#### Being smart with resources
**Resource slicing** 需要我们一小部分简单的工作就能实现。如果使用Resource slicing，则要保证我们的资源都被Asset Catalogs管理。在Xcode 7中，能够标记资源被使用设备的 **Memory** 和 **Graphics** ，如下：
![](/images/2016.01.04.07.png)

##### Your first fix
在开始的时候介绍过Santa Cruz PNGs这个文件因为被放到Main bundle中，所以不能被编译进入到Assets.car，进而也不能使用Resource slicing。下面看一下我们怎么修改，使其能够使用：
![](/images/2016.01.04.08.png)

选择New Image Set后，将新加入的set命名为Santa Cruz，紧接着做如下操作：
![](/images/2016.01.04.09.png)  
> 纠正一下 上图左边的内容应该是删除，包括在Finder内也应该删除   

然后在不同的设备上运行App，最后发现Asset.car文件的大小并不一致。这个是因为在安装的时候，会根据设备安装对应的资源。   

##### Lazily (down)loading content
苹果提供On-Demand Resources技术，简称ODR。ODR允许你将资源存储在苹果的服务器上，然后在你App使用的时候再去下载。NSBundleResourceRequest是处理ODR的类，使用这个类能够通过tag下载对应的资源。images, data, OpenGL shaders, SpriteKit Particles, Watchkit Complications等都可以使用ODR。

##### Wire things up to use tags
下面我们修改代码，实现资源的下载，修改MapChromeViewController.swift对应方法如下：
{% codeblock lang:swift %}
  private func downloadAndDisplayMapOverlay() {
//    displayOverlayFromBundle(NSBundle.mainBundle())
    guard let bundleTitle = mapOverlayData?.bundleTitle else {
      return
    }

    let bundleResource = NSBundleResourceRequest(tags: [bundleTitle])

    bundleResource.beginAccessingResourcesWithCompletionHandler { [weak self] (error) -> Void in
      NSOperationQueue.mainQueue().addOperationWithBlock({ () -> Void in
        if error == nil {
          self?.displayOverlayFromBundle(bundleResource.bundle)
        }
      })
    }

  }
{% endcodeblock %}    

这时候我们运行代码，可能会在控制台输出错误，这是因为我们对应的bundle并没有tag，我们需要给bundle添加tag：
![](/images/2016.01.04.10.png)

然后我们重新编译运行我们的程序，然后按照上面的查看编译运行的程序的大小，发现小了许多。对比之前的编译生成的文件，发现运行文件里面不包含bundle了。
![](/images/2016.01.04.11.png)  

>如果你的App在App Store上可能这个资源文件下载的很慢。但是在开发的过程中，Xcode会利用本地网络作为服务器，然后在设备上能够下载到，所以在开发的过程中如果电脑关了，那ODR也就不能使用了。   

#### Make it download faster   
在我们使用ODR的过程中，如果bundle比较大，可能再下载的过程中就会比较耗时，并且在下载过程中用户不知道，这样用户体验就不好。我们可以再Resource下载的过程中给用户一些提示，修改下面的代码：
{% codeblock lang:swift %}
// add 为新添加的 ProgressView是程序已经添加上的
private func downloadAndDisplayMapOverlay() {
//    displayOverlayFromBundle(NSBundle.mainBundle())
  guard let bundleTitle = mapOverlayData?.bundleTitle else {
    return
  }

  let bundleResource = NSBundleResourceRequest(tags: [bundleTitle])

  bundleResource.loadingPriority = NSBundleResourceRequestLoadingPriorityUrgent  //add

  loadingProgressView.observedProgress = bundleResource.progress // add

  loadingProgressView.hidden = false // add
  UIApplication.sharedApplication().networkActivityIndicatorVisible = true // add

  bundleResource.beginAccessingResourcesWithCompletionHandler { [weak self] (error) -> Void in
    NSOperationQueue.mainQueue().addOperationWithBlock({ () -> Void in
      self?.loadingProgressView.hidden = true // add
      UIApplication.sharedApplication().networkActivityIndicatorVisible = false // add
      if error == nil {
        self?.displayOverlayFromBundle(bundleResource.bundle)
      }
    })
  }

}
{% endcodeblock %}

> 如果用户已经下载过某个bundle，下次在使用的时候就不会再去下载了。

#### The many flavors of tagging
虽然添加了ProgressView，在体验是好了一点，但是需要注意测试的时候是使用的本地的网络，所以比较快，但是如果要是提交到App Store上，那可能下载就是比较慢了，如果再配上用户没有WiFi那可能就没法用了，所以我们还需要做其他的一些调整。

##### Initial install tags  
使用Initial install tags，我们可以设置哪些bundle会在我们App初始化安装的时候就会被下载。 下面下介绍一下ODR三种下载的时机吧：
* **Initial Install Tags** 在ipa下载的时候一同下载
* **Prefetched Tag Order** 在程序下载完成后，下载对应的资源，然后按顺序排列。
* **Prefetched Tag Order** 按需下载   
下面是配置的地方：
![](/images/2016.01.04.12.png)  

#### Purging content
应用程序在使用的过程中通过ODR下载了对应的bundle，但是有时候我们需要清理一些已经下载过的并且不使用的bundle。在介绍怎么删除之前先看一下怎么查看下载的ODR：
![](/images/2016.01.04.13.png)  

##### Set a resource to be purged   
在MapChromeViewController.swift添加如下代码：
{% codeblock lang:swift %}
  // new add 是新加的代码
  var overlayBundleResource: NSBundleResourceRequest? // new add
  private func downloadAndDisplayMapOverlay() {
//    displayOverlayFromBundle(NSBundle.mainBundle())
    guard let bundleTitle = mapOverlayData?.bundleTitle else {
      return
    }

    let bundleResource = NSBundleResourceRequest(tags: [bundleTitle])
    overlayBundleResource = bundleResource // new add

    bundleResource.loadingPriority = NSBundleResourceRequestLoadingPriorityUrgent  //add

    loadingProgressView.observedProgress = bundleResource.progress // add

    loadingProgressView.hidden = false // add
    UIApplication.sharedApplication().networkActivityIndicatorVisible = true // add

    bundleResource.beginAccessingResourcesWithCompletionHandler { [weak self] (error) -> Void in
      NSOperationQueue.mainQueue().addOperationWithBlock({ () -> Void in
        self?.loadingProgressView.hidden = true // add
        UIApplication.sharedApplication().networkActivityIndicatorVisible = false // add
        if error == nil {
          self?.displayOverlayFromBundle(bundleResource.bundle)
        }
      })
    }

  }

  // new add
  override func viewDidDisappear(animated: Bool) {
    super.viewDidDisappear(animated)
    // 告诉系统结束了对资源的访问
    overlayBundleResource?.endAccessingResources()
  }
{% endcodeblock %}

上面的代码，我做测试的时候不清楚会在什么时候会删除，我也模拟了内存警告，如果谁清楚，还请告诉我，谢谢。

坚持了好几天中午写完了，这篇笔记，一篇笔记13张截图，好累。
