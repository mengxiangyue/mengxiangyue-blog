title: OC项目中使用Swift
date: 2016-01-11 21:37:38
categories:
  - iOS Tips
tags:
  - iOS 9
---

最近公司的项目开始计划使用Swift，由于原先的工程都是使用OC编写的，不可能一下全部转换成Swift，所以采用OC与Swift混编的方式是最好的选择。这篇文章只是一个简单的介绍，并没有太高深的知识。

我新建了一个演示的OC工程，当然你可以使用你已经存在的OC的工程。如果我们想要在OC工程中使用Swift的代码，Swift的代码默认是使用module管理的，同样这里我们也需要把我们的Swift代码作为一个module暴露给我们的OC工程，修改下面的配置：
![](/images/2016.01.11/01.png)

上面的修改了一个配置项，有一个Product Module Name在后面会使用。

<!---more---->

在工程里面点击File/New/File...,选择iOS/Source/Cocoa Touch Class,按照如下填写创建一个新的文件：
![](/images/2016.01.11/02.png)

上图中的Subclass of一定要设置为NSObject或其子类，否则OC工程将不会找到该类。   

点击确认后会选择保存路径，点击Create，出现如下界面：
![](/images/2016.01.11/03.png)

> 这个界面是询问是否创建桥接的头文件，这个文件在Swift调用OC代码的时候比较管用，但是在OC中调用Swift的时候我发现没有什么卵用。   

选择Don't Create按钮。   

在Test.swift中添加如下的代码(解释都在注释里面了)：
{% codeblock lang:swift %}
import UIKit

/*
    如果Swift类想要被OC发现，必须继承自NSObject并且使用public标记，并且该类中想要被OC访问的方法也必须使用public标记，具体知识可以去看Swift的访问控制
    原因：Swift的代码对于OC来说是作为一个module存在的

    当然全局的Swift函数，我还没发现怎么在OC中访问，如果哪位清楚还请告诉一下，谢谢！
*/


public class Test: NSObject {
    public func log() {
        print("这是Swift的方法")
    }
}

public func globalLog() {
    print("这是Swift全局的log方法")
}
{% endcodeblock %}   

我们在我们想要调用Swift类的方法里面引入头文件："Product Module Name-Swift.h",其中Product Module Name替换成在上面配置项中显示的内容，例如：
{% codeblock lang:swift %}
#import "ViewController.h"
// 引入Swift头文件
#import "OCAndSwift-Swift.h"

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];

    // 使用Swift的类
    Test *test = [[Test alloc] init];
    [test log];
}

@end
{% endcodeblock %}  

这样我们就能够在OC中使用Swift的代码了，最后还要说明一点："Product Module Name-Swift.h"（例子中的是OCAndSwift-Swift.h），是由编译器自动生成的，如果import后没有提示，编译一下。并且只有在工程中包含至少一个Swift文件的时候，才会自动生成这个文件，所以如果工程中如果没有Swift文件的时候，就算在配置中设置对了，import该文件也会报错。
