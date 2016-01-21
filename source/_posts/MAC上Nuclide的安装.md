title: MAC上Nuclide的安装
date: 2016-01-21 23:10:24
categories:
  - React Native
tags:
  - React Native
---

Nuclide是Facebook开发的开发React Native的开发工具，基于Github的Atom开发，以Atom插件的形式存在。在MAC版Atom安装插件可以使用系统自带的插件安装方式安装：Atom->Preferences..->Install,搜索Nuclide-installer,点击install就可以安装Nuclide了。安装完了是不是很幸福，能开心的编程了。啊啊啊啊啊.....但是事实并非如此，打开编辑器发现根本不能用，卡成翔了，查看一下进程，发现有个AtomHelper CPU占用率超过100%了。去github查看issue发现好多人都遇到了这个问题。自己试验了半天找到了一种安装方式：

1. 删除已经安装的Nuclide插件    
  这里我是直接卸载Atom，这样能够删除安静。对于曾经安装的插件，记录下来，重新安装。
2. 编译Nuclide   
  从<https://github.com/facebook/nuclide>下载Nuclide，终端进入下载后的目录，执行命令 **./scripts/dev/setup** 。编译的过程中如果没有出现错误信息，就表示编译成功了。将文件夹重命名为nuclide，然后拷贝到~/.atom/packages/目录下。重启Atom，第一次启动应该比较慢，等启动结束后进入Atom->Preferences..->Packages，如果列出了nuclide，表示安装成功了。  
3. 升级flow   
  这样安装后可能flow不能使用，因为.flowconfig文件末尾会有一个版本，如果我们本地版本低于其中配置的版本将不能使用flow，我们可以直接删除这个版本配置，这样就能使用了。但是这个并不是好的解决方案，好的解决方案是将flow升级到最新版本。在终端中执行如下命令：
```shell
brew update
brew upgrade flow
```  

升级后应该flow就能正常使用了。   

这样就能够正常的使用Nuclide了。
