title: iOS9 by Tutorials 学习笔记四：APP 瘦身
date: 2016-01-04 22:26:09
categories:
  - iOS 9 by Tutoials
tags:
  - iOS 9
---

>这篇文章在书中的标题是App Thinning，这里我给翻译成了App 瘦身。   

iPhone经过这几年的发展，已经发生了很大的变化，例如屏幕变得更加多样，尺寸更多，内存变得更大，CPU的架构也在变化。伴随着iPhone的变化，iOS也在变化，例如AutoLayout、size classes、split view controller等。同样这些技术及设备的变化给我在开发的过程中也造成了许多的问题，但是苹果通过不断推出新的技术，努力在帮助我们使用同一套代码开发适应多个设备的Universal的App。但是Universal App虽然在开发的过程中，方便了我们开发人员，但是对于用户来说就不那么友好了，由于需要适配多种设备，所以里包含所有设备的代码，但是真正的在运行的时候，我们并不需要那么多相关的代码及资源。    

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
![](/images/2016.01.04.04.png)   

上图中标注的说明如下：
1. Assets.car是Assets.xcassets被Xcode进行编译后的文件。
2. Old CA Maps是真实运行在设备上的可执行文件。
3. Santa Cruz PNGs 这个是图片文件，但是没有被编译到Assets.car文件中，这是因为它并没有放到Assets.xcassets中，而是放到了工程的顶层文件中。
4. SD_Map.bundle 这个就是地图图片文件，但是奖金120MB。

#### Measuring your work


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
