title: Hexo结合Github Pages搭建静态博客
categories:
  - 工具
tags:
  - Github
  - Hexo
date: 2015-12-19 13:17:00
---
已经好久没有写过博客了，主要是因为懒了。  
前几天无聊点进了阿里云的广告里面，无意中看到了域名查询，查了一下自己的名字的域名，发现还没有注册（我原来记的这个域名是已经注册的了），然后就花钱买下了这个域名。然后因为这个买到的域名引出了了后面搭建博客的这么一堆事。  

我自己已经有一个博客了，是在CSDN的博客，博客地址：<http://blog.csdn.net/mengxiangyue>,那个博客维护了好久了，从大三开始吧。断断续续也写了好多年了，里面的文章我个人感觉水平也就一般。但是因为这些水平一般的文章，我也收获了很多，首先是收获了一个笔记吧，然后是申请成功了CSDN的博客（伪）专家，因为这个专家的身份，有时候会参加一些CSDN举办的活动。说到写博客，我个人感觉最重要的是技术的积累，我也跟很多人说过写博客这个事情，但是貌似听的人不是特别多。  

好了貌似扯得有点远了，下面进入正题。  
上面我已经说过我买了一个域名，那就大概梳理一下提纲吧：  
>1. 购买一个域名
>2. 在github上面生成一个github pages的仓库
>3. 搭建Hexo环境
>4. 配置博客
>5. 部署博客

在开始介绍步骤之前，先扯几句Hexo、Github Pages的东西（我也不是完全了解，只是我的理解，如果哪里错了，还请指出）  Github Pages是Github提供的一个静态网站的功能，可以根据在Github仓库的HTML、CSS、js文件生成一个网站，然后提供一个二级域名，可以直接访问。这里说的静态网站，就是所有的页面的HTML页面都是静态的、已经生成好的，而不是动态生成的。   
Github Pages使用的是已经生成好的HTML，如果我们自己手动写HTML会累死的，所以就需要使用工具来生成。搭配Github Pages的比较不错的工具有jekyll、Hexo等，查了一下资料说jekyll比较复杂，Hexo比较简单，最后选择了后者。

下面按步骤说吧：
### 1. 购买一个域名  
>这里如果你不想使用独立域名，也可以略过这一步    

注册、登录阿里云账号，然后选择->域名与网站服务，查询自己想要的域名，加入清单，结算这样就买了一个域名。这里先不配置DNS，后面会说。  

### 2. 在github上面生成一个github pages的仓库
2.1 在github上创建一个仓库,名字你自己随便起就可以了。如图： ![](/images/2015.12.19.01.png)  
2.2 创建完了后，选择该仓库的Settings，然后找到Github Pages部分。![](/images/2015.12.19.02.png)   
点击了之后会进入选择主题，这时候随便选择，然后点击发布就可以了。做完了这些之后，我们可以访问以下http://<你的Github用户名>.github.com，这时候如果能够打开说明成功了。   

### 3. 搭建Hexo环境
3.1 安装npm、nodejs环境，这个自己百度吧，我就不写了。   
3.2 安装Hexo   
    执行如下命令：
```shell
npm install hexo-cli -g # 安装hexo工具
```
3.3 初始化博客   
```shell
hexo init blog #初始化一个blog 可以cd到你想要生成博客的目录
cd blog # 切换到创建博客的目录下
npm install # 安装nodejs依赖 注册这里一定要在init后面执行一次这个，否则可能会出现一些未知的错误
```
这时候生成的博客目录：![](/images/2015.12.19.03.png)   
执行如下命令：    
```shell
hexo server
```
这时候在浏览器访问<http://0.0.0.0:4000/>，应该能够看到已经搭建好的博客了。  

### 4. 配置博客
这里主要配置主题、评论插件多说、RSS、域名。  
我使用的主题是jacman，详细的介绍在[这篇文章](http://wuchong.me/jacman/2014/11/20/how-to-use-jacman/)已经介绍了，我这里只是说了一下我自己配置过程中的一些注意的地方。我的博客的源文件也已经开源了，如果有不明白的地方可以下载看一下，地址<https://github.com/mengxiangyue/mengxiangyue-blog>   
每个配置项的后面留一个空格，然后再写配置的值，如下"首页:"与后面的"/"之间要留一个空格，否则会出现问题。
```
首页: /
```
如果一个配置项目包含多个子项目，子项目起始位置要留空格，如下：
```
imglogo:
  enable: true  
  src: img/logo.png
```
另外在配置的过程中，可能会出现看着配置项目没问题，然后就是出现错误，这时候可以试试换一个工具打开配置文件，然后配置，可能有些工具的编码问题造成的。   
在配置的过程中涉及到图片的路径都在themes/jacman/source/img目录下面。   

#### 配置多说插件
注册多说（<http://duoshuo.com/>）账户，然后添加站点，按照自己的要求填写信息。![](/images/2015.12.19.04.png)  
右上角点击你新建的使用多说的配置站点，然后看浏览器地址栏的地址，如果出现admin结尾，然后记录下来多说前面的用户名，比如我的是http://mengxinagyue.duoshuo.com/admin/ ，然后我的用户名就是mengxiangyue。找到配置文件在对应的地方改成你自己的用户名   
```
duoshuo_shortname: mengxinagyue  #修改成你自己的用户名
```
到这里多说配置完了。
#### 配置RSS
执行如下命令：
```
npm install hexo-generator-feed --save
```
在博客的配置文件_config.yml中添加如下配置：
```
Plugins:
  hexo-generator-feed

#Feed Atom
feed:
  type: atom
  path: atom.xml
  limit: 20
```

执行如下命令：
> 这里先去创建一篇测试文章，因为多说插件只有在文章中才能看到，怎么创建文章，这个去看官方文档吧。

```
hexo generate  # 重新生成配置文件 保证我们的修改会生效
hexo server
```
访问<http://0.0.0.0:4000/>，配置应该已经生效

#### 配置域名
安装阿里云的帮助文档，进入到域名解析配置页面，然后选择CNAME进行解析（这里的原理我也没有详细了解过），类似下面的配置（图中的域名修改为你自己的），然后保存。  
![](/images/2015.12.19.04.png)  
> 配置完后可能立即访问也没有效果，需要过一会才会生效，这个涉及到DNS的知识，请自己百度吧。  

在你的博客的文件夹的source目录下创建一个"CNAME"文件，没有后缀，里面的内容就只是写上你的域名就可以了。

### 5. 部署博客
在命令行执行如下命令，安装hexo-deployer-git,这个主要适用于将博客部署到Github上的工具。
```
npm install hexo-deployer-git --save
```
在博客的配置文件_config.yml中添加如下配置：
```
deploy:
  type: git
  repo: <你的博客的仓库地址，即查看仓库时候浏览器地址栏的地址>
```
最后执行如下命令：
> 在部署的过程中可能会需要输入用户名密码，如果还是不行可能还需要配置SSH，因为我的电脑原来早就已经配置过了，所以这里不清楚。  

```
hexo deploy
```
最后出现部署成功的提示，这时候访问你的博客应该就能看到最新的了。

### 下面是我遇到的一些问题  

1 about路径不存在    
 jacman主题上菜单栏里面有一个about菜单项目，它指向的地址是about/目录，我们可以使用如下命令创建该目录，然后修改里面的index.md文件。     

**hexo new page "about"**   

2 图片路径的问题
我们可以在source目录下创建一个images目录，然后在使用的时候，使用相对路径，例如：'![](/images/2015.12.19.05.png)'  

3 站内搜索
配置了半天的百度搜索，只能说自己能力有限，最后懒得弄了，就没弄，如果有谁清楚，还请赐教。  

4 目录序号错误  
Hexo如果开启文章目录，会根据Markdown的##标记自动生成文章目录，并且自动添加序号，但是如果我们的文章中也使用了序号，那就会出现两个序号，如图：   
![](/images/2015.12.19.06.png)   
我解决这个问题是是通过js删除了序号，因为我对于nodejs不熟悉，所以不能从那上面改只能想其他的方法了。在博客的themes/jacman/layout/\_partial/after_footer.ejs文件中添加如下代码，代码位置可以参看我的Github工程：  

```javascript
<!-- 解决自动生成文章目录序号问题 -->
<script type="text/javascript">
var regex = new RegExp("^\\d+\\.\\d* ");
var tocItemTextArray = $(".toc-article .toc-item .toc-text");
for (var i = 0; i < tocItemTextArray.length; i++) {
  var item = tocItemTextArray[i];
  item.textContent = item.textContent.replace(regex, "");
}
</script>
```


5 回到顶部不显示   
jacman主题默认是滚动距离超过1000才会显示回到顶部按钮，如果文章过短将永远不会显示，我这里改成了300，可以在themes/jacman/source/js/totop.js中修改如下属性为300：   

```
var upperLimit = 300;
```

终于把这个写完了好费劲。如果有什么问题可以找我交流。
