title: iOS9 by Tutorials 学习笔记一：Swift 2.0
date: 2015-12-22 22:15:07
categories:
  - iOS 9 by Tutoials
tags:
  - iOS 9
---

Apple在前段时间开源了Swift，在iOS开发领域中又制造了一阵骚动，看了一眼Swift的开发路线图，计划在明年的秋天发布Swift 3.0。Apple现在在Swift上变得也更加的开发，鼓励社区贡献代码，也开始接纳社区的一些反馈了。苹果改变以往的封闭的姿态，表明了它对于Swift语言的重视，同时也说明了Swift语言苹果会加大力度去优化，所以现在对于我们iOS开发人员来说，是时候开始学习iOS了。   
前段时间也面试了几个人，简历里面好几个都写了精通Swift，但是一问问题好多都答不上来，简历上真的。。。。。更多的人貌似没有开始学Swift，但是最后我都建议他们去学习一下Swift。    
扯远了，回到正题，这篇文章是我的学习笔记，非本人原创内容，只是在看《iOS 9 by Tutorials》这本书时候的一些笔记，然后加上自己的一些理解而已。    

Swift 2中加入了几个（作者认为）比较重要的改进，如下：
* 新的控制流
* （相对）完善的错误处理模型
* 协议扩展
* 模式匹配的增强
* API可用性检测
* 其他一些。。。。。。

### 控制流   
在书中首先作者解释了一下控制流，感觉不错：程序中任何能够影响程序执行到不同的路径的结构或者关键字都可以叫做控制流，原文：any construct or keyword that causes the execution of your program to follow a different path can be considered "control flow".   
#### repeat...while
repeat...while是重复的意思，类似于其他语言中的do...while。其实在Swift 1.x中还是使用的do...while，在2.x中为了与do...catch区分，所以改成了repeat，但是语义上还是没有变化。这里多说一句，Swift的好多改进，都是为了让程序读上去更加明确，例如Optional、guard等也有这方面的考虑。   
> 本例子中的代码都是在Playground中实现的    

{% codeblock lang:swift %}  
var x = 2
repeat {
    print("x:\(x)")
    x += 1 // Swift计划在3.0中移除 ++ -- 所以还是尽量少用吧
} while x < 10 // 这个地方可以添加括号
{% endcodeblock %}
上面while后面可以不适用括号，这个也是Swift的一个改进，Swift中只有必要（即语义不明确）的时候才会要求必须加括号。

<!--more-->

#### guard
guard这个词我也不知道怎么翻译，这里就不翻译了。但是这个关键字的作用的就是一个先决条件的检测。先看下面的例子：   

{% codeblock lang:swift %}
func printName(name: String) {
    guard !name.isEmpty else {
        print("no name")
        return
    }
    print(name)
}
printName("")
printName("MengXiangYue")
{% endcodeblock %}  

上面的例子是一个没有意义的例子，只是为了演示。定义了一个函数打印传入的名字，这个函数的要求如果传入的name为空，就判定程序错误，然后返回不执行代码。**guard** 后面跟一个条件，条件为真的时候不会执行else，当条件为假的时候将会执行else，这样就能够达到了我们的要求。但是可能又回说，我用一个if-else也能够实现这个功能，但是如果要是跟Optional结合在一起就比if-else方便多了，下面继续看这个例子：  

{% codeblock lang:swift %}
func printName(inName: String?) { // 这里变成了可选值了
    guard let name = inName else {
        print("no name")
        return
    }
    guard !name.isEmpty else {
        print("no name")
        return
    }
    print(name)
}
printName("")
printName("MengXiangYue")
{% endcodeblock %}    

上面的例子中传入的参数是一个可选值，这时候使用『guard let name = \_name else...』,这个类似于if let解包的方式，但是看下面我们使用guard声明的name变量，在下面是能够正常使用的，但是考虑如果使用if let这个就不能使用了，所以我认为guard结合Optional是使用起来最方便的。另外这个东西也可以实现类似NSAssert类似的功能，只是这个不会崩溃。

### （相对）完善的错误处理模型
这里我加了一个相对，主要是指的相对于Swift 1.x，2.x的错误处理好用了不少，但是相比于java等其他部分语言，还是不完善，Swift中的错误处理，对于抛出错误来说，你只是知道该函数抛出了错误，但是不清楚这个函数抛出了什么错误，书中有句话写的很正确，这个要求写程序的时候一定要在文档中写明，会抛出的各种异常（在java中会明确的抛出Exception，Exception与Swift的Error功能一致）。    

另外相对于Objective-C的NSError把指针传递进去，然后等函数执行完成之后检查，已经先进了不少，鼓掌。。。。。   
定义下面的一个协议：   

{% codeblock %} swift
protocol JSONParsable {
    static func parse(json: [String: AnyObject]) throws -> Self
}
{% endcodeblock %}  

这个协议定义了一个静态方法，这里不能叫做类方法，以为协议同时可以应用到Struct上，可以叫类型方法。这个函数使用了**throws** 关键字，这个关键字表示该方法可能会抛出一个错误，这里也看不出来抛出什么错误（你妹啊，啥错误都不知道），所以就更加突出这时候注释的重要性（可以写篇文章：论注释的重要性，哈哈哈）。   

那既然说到抛出错误，那我们就得定义错误，在Swift中定义错误比较容易，只要定义一个枚举类型，然后遵守**ErrorType** 协议就可以了。OC中的NSError同样也实现了**ErrorType** 协议，所以我们能够在OC和Swift中使用NSError没有问题。下面定义一个错误：

{% codeblock lang:swift %}
enum ParseError: ErrorType {
    case MissingAttribute(message: String)
}
{% endcodeblock %}

定义一个错误比较简单，跟普通的枚举没什么不同，这里定义了一个有关联值的枚举。关联值这里要多扯一句，关联值这个东西在Swift中能够解决好多与类型相关的东西，有时候我们经常会遇到某个类型与值相关，比如我们自己的工程中，网络请求错误需要带着错误码和错误提示，这时候我在OC中可能需要返回三个参数，但是在Swift中我可以只是返回一个枚举，然后关联上另外的两个值。对于多个有关系的值，同样也可以使用元组，曾经看kingfisher的时候，作者把一个类的配置参数都放到一个元组里面，然后解析这个元组，这样参数可能更加清晰。   
又扯远了，回到正题。下面我们实现一个结构体Person：

{% codeblock lang:swift %}
struct Person: JSONParsable {
    let firstName: String
    let lastName: String

    static func parse(json: [String : AnyObject]) throws -> Person {
        guard let firstName = json["first_name"] as? String else {
            let message = "Expected first_name String"
            throw ParseError.MissingAttribute(message: message) // 1
        }

        guard let lastName = json["last_name"] as? String else {
            let message = "Expected last_name String"
            throw ParseError.MissingAttribute(message: message) // 2
        }
        return Person(firstName: firstName, lastName: lastName)
    }
}
{% endcodeblock %}  

代码比较简单就不过多解释了，就是在不同情况下抛出不同的异常。我们在调用这个方法的时候，需要处理这些异常，这时候就使用到了Swift中的do...catch。下面是代码：

{% codeblock lang:swift %}
do {
    let person = try Person.parse(["foo": "bar"])
} catch ParseError.MissingAttribute(let message) {
        print(message)
} catch {
        print("Unexpected ErrorType")
}
{% endcodeblock %}    

do后面需要使用{}将抛出异常的函数包起来，调用抛出异常的方法的时候，需要使用try关键字，然后后面跟着需要捕获的异常，如果清楚需要捕获的异常的类型，可以再catch后面加上异常类型，如果没有异常类型，那表示捕获所有的异常。异常会按照catch的顺序挨个匹配，直到找到第一个匹配的结束。   

如果我们对于异常不关心，我们可以使用try?、try!调用方法，其中try?调用方法会返回一个Optional值，如果调用成功将会返回对应的结果，如果失败则返回nil，程序一定不会崩溃，但是如果我们直接使用try!如果有异常抛出，程序将会崩溃。所以只有在保证我们调用的函数不会抛出异常的时候才能使用try!。  

{% codeblock lang:swift %}
let p1 = try? Person.parse(["foo": "bar"])  // nil
let p2 = try! Person.parse(["first_name": "Ray", "last_name": "Wenderlich"]) // Person
let p3 = try! Person.parse(["foo": "bar"]) // error crash
{% endcodeblock %}

### 协议扩展
在这一部分使用一个例子来介绍协议扩展，协议扩展是在Swift 2.x中一个比较重要的思想。详细的可以看看WWDC 2015 Session 408了解。下面定义一个验证字符串规则的一个协议：  

{% codeblock lang:swift %}
protocol StringValidationRule {
    func validate(string: String) throws -> Bool // 验证是否合法的方法
    var errorType: StringValidationError { get }  // error的类型
}
{% endcodeblock %}  

上面定义了校验规则的协议，下面定义一个校验器协议：  

{% codeblock lang:swift %}
protocol StringValidator {
    var validationRules: [StringValidationRule] { get }
    func validate(string: String) -> (valid: Bool, errors: [StringValidationError])
}
{% endcodeblock %}   

StringValidator这个校验器，有一个保存校验规则的数组，然后有一个校验方法，返回一个元祖，包含最终的校验结果，及错误。这里我们考虑一下对于校验器可能我们处理的逻辑都是一样的，就是循环所有的校验规则，然后查看是否校验成功。这个逻辑算是比较一致，如果我们把这个放到每个实现该协议的类型里面，那代码可能会重复。这时候我们可以提供一个默认的实现，这就是协议扩展（类似于虚函数的功能）。   

{% codeblock lang:swift %}
extension StringValidator {
    func validate(string: String) -> (valid: Bool, errors:[StringValidationError]) {

        var errors = [StringValidationError]()
        for rule in validationRules {
            do {
                try rule.validate(string)
            } catch let error as StringValidationError {
                errors.append(error)
            } catch let error {
                fatalError("Unexpected error type: \(error)")
            }
        }
        return (valid: errors.isEmpty, errors: errors)
    }
}
{% endcodeblock %}

下面我们实现一个字符串以某些字符开始和以某些字符结束的的规则。首先定义一下上面的StringValidationError
{% codeblock lang:swift %}
// 错误类型
enum StringValidationError: ErrorType {
    case MustStartWith(set: NSCharacterSet, description: String)
    case MustEndWith(set: NSCharacterSet, description: String)
    var description: String {
      let errorString: String
      switch self {
      case .MustStartWith(\_, let description):
        errorString = "Must start with \(description)."
      case .MustEndWith(\_, let description):
        errorString = "Must end with \(description)."
      }
      return errorString
    }
}   

// 扩展String
extension String {
    public func startsWithCharacterFromSet(set: NSCharacterSet) -> Bool {
        guard !isEmpty else {
            return false
        }

        return rangeOfCharacterFromSet(set, options: [], range: startIndex..<startIndex.successor()) != nil
    }

    public func endsWithCharacterFromSet(set: NSCharacterSet) -> Bool {
        guard !isEmpty else {
            return false
        }

        return rangeOfCharacterFromSet(set, options: [], range: endIndex.predecessor()..<endIndex) != nil
    }
}

struct StartsWithCharacterStringValidationRule : StringValidationRule {

    let characterSet: NSCharacterSet
    let description: String
    var errorType: StringValidationError {
        return .MustStartWith(set: characterSet, description: description)
    }
    func validate(string: String) throws -> Bool {
        string
        if string.startsWithCharacterFromSet(characterSet) {
            return true
        } else{
            throw errorType // 4
        }
    }
}

struct EndsWithCharacterStringValidationRule: StringValidationRule {
    let characterSet: NSCharacterSet
    let description: String
    var errorType: StringValidationError {
        return .MustEndWith(set: characterSet, description: description)
    }
    func validate(string: String) throws -> Bool {
        if string.endsWithCharacterFromSet(characterSet) {
            return true
        } else {
            throw errorType
        }
    }
}
{% endcodeblock %}

两个验证规则创建好了，下面我们创建一个校验器：
{% codeblock lang:swift %}
// 这个校验器实现了StringValidator，但是由于StringValidator存在扩展，所以可以不用实现该协议中的func validate(string: String) -> (valid: Bool, errors:[StringValidationError])方法
struct StartsAndEndsWithStringValidator: StringValidator {
  let startsWithSet: NSCharacterSet
  let startsWithDescription: String
  let endsWithSet: NSCharacterSet
  let endsWithDescription: String
  var validationRules: [StringValidationRule] {
    return [
      StartsWithCharacterStringValidationRule(characterSet: startsWithSet, description: startsWithDescription),
      EndsWithCharacterStringValidationRule(characterSet: endsWithSet, description: endsWithDescription)
    ]
  }
}

// 下面使用一下
et numberSet = NSCharacterSet.decimalDigitCharacterSet()
let startsAndEndsWithValidator = StartsAndEndsWithStringValidator(startsWithSet: letterSet, startsWithDescription: "letter", endsWithSet: numberSet, endsWithDescription: "number")

startsAndEndsWithValidator.validate("1foo").errors.description
{% endcodeblock %}   

上面的内容是一个简单的例子，我将书中的例子做了一些简化。   

下面我们再看一个例子，在扩展协议的时候我们可以结合where关键字，使符合where条件的类型，才会自动的存在默认的协议扩展。
{% codeblock lang:swift %}
// 扩展了MutableCollectionType协议，这个协议仅对Index为Int类型的实现了MutableCollectionType的类型生效  
// Index是定义在MutableCollectionType的父协议MutableIndexable中的关联类型
extension MutableCollectionType where Index == Int {
  // 该方法任意的交换集合元素
  mutating func shuffleInPlace() {
    let c = self.count
    for i in 0..<(c-1) {
      let j = Int(arc4random_uniform(UInt32(c - i))) + i
      guard i != j else { continue }
      swap(&self[i], &self[j])
    }
  }
}

var people = ["Chris", "Ray", "Sam", "Jake", "Charlie"]
people.shuffleInPlace()
{% endcodeblock %}

### 模式匹配的增强   
在Swift中可以不仅可以再实现协议扩展的时候使用，还可以在for循环，也可以在if-let、switch、if-case的使用，如下例子：
{% codeblock lang:swift %}
let names = ["Charlie", "Chris", "Mic", "John", "Craig", "Felipe"]
var namesThatStartWithC = [String]()
// 将以"C"开头的名字，加入到数组namesThatStartWithC中
for cName in names where cName.hasPrefix("C") {
  namesThatStartWithC.append(cName)
}

// 定义一个Author
public struct Author {
    public let name: String
    public let status: Additional_Things_PageSources.AuthorStatus
    public init(name: String, status: Additional_Things_PageSources.AuthorStatus)
}
let authors = [
  Author(name: "Chris Wagner", status: .Late(daysLate: 5)),
  Author(name: "Charlie Fulton", status: .Late(daysLate: 10)),
  Author(name: "Evan Dekhayser", status: .OnTime)
]
var slapLog = ""
for author in authors {
  if case .Late(let daysLate) = author.status where daysLate > 2 {
    slapLog += "Ray slaps \(author.name) around a bit with a large trout \n"
  }
}
{% endcodeblock %}

### API可用性检测
在Swift 2.x中检测某个API是否可用，不用像原来一样判断是否能够响应某个API，直接使用如下代码，使其在该版本系统下生效即可：
{% codeblock lang:swift %}
if #available(iOS 9.0, \*) {
  // 调用在iOS 9下才能使用的API
}
{% endcodeblock %}

### defer关键字
defer在Swift中表示，在方法结束的时候一定会调用的代码。在程序中我们经常将一些内存回收、状态回复等动作放在代码的最后，但是如果在前面代码执行的过程中，发生了异常，那么可能后面的代码就不能执行，造成程序错误。但是使用defer关键字，能够保证不管程序是否正常结束，该代码一定会被执行。   

例如在使用ATM的时候，不管使用的过程中发生了什么异常都必须保证最后必须把银行卡退给用户，这个在这里使用defer关键字就比较合适。

{% codeblock lang:swift %}
struct ATM {
  mutating func dispenseFunds(amount: Float, inout account: Account) throws{
   defer {  // 保证一定能够退卡成功
     log += "Card for \(account.name) has been returned to customer.\n"
     ejectCard()
   }
   // 其他的逻辑处理
 }
  func ejectCard() {
    // physically eject card
  }
}
{% endcodeblock %}   

终于是把这篇文章算是写完了，后面的一部分都是一些小的知识点，慢慢积累吧，自己的读书笔记，希望对别人有帮助吧。
