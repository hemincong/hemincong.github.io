---
layout: post
title: Swift笔记-ErrorType
permalink: /Swift-ErrorType/
---

Swift笔记-ErrorType
======

最近忙一个项目,一个人要和三个人的效率比赛。最后看得上是貌似要撑不下去的样子,挺烦心。

也因为忙，没怎么写，而且最近Swift的改动，慢慢研究与实用了一点，写两篇当笔记吧。

Swift 2 有几个比较引人注意的新特性，现在说说ErrorType，按惯例先给[官方文档](https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/Swift_Programming_Language/ErrorHandling.html)

别人的博客说了的，都不说了

ErrorType定义与处理
======
首先，ErrorType什么都不是，请看定义:

{% highlight swift%}
public protocol ErrorType {
}

extension ErrorType {
}
{% endhighlight %}

{% highlight swift%}
enum ETeGongError: ErrorType, CustomStringConvertible {
    case SeverResonpseCannotParse
    case SeverResonpseNoRetCode
    case CommonServerError(code: Int, description : String)
    case NeedLogin

    var description: String {
        switch self {
        case .SeverResonpseCannotParse:
            return "系统返回错误，请联系客服"
        case .SeverResonpseNoRetCode:
            return "系统返回错误，请联系客服"
        case CommonServerError(_, let description):
            return description
        case .NeedLogin:
            return "请先登录"
        }
    }
}
{% endhighlight %}

留意ErrorType也可以多种构造。在enum里面顺便就做好了description，这也是Swift方便的地方，无需要另外再处理

下面是一个例子的处理错误的方法，我就写在对应的ViewController的Extension里面做统一的处理，留意里面的case let xx as xx的方法
我也是乱撞出来的，实际上是switch case的key binding，但为什么能用as 就估计是看漏了书了
留意as ETeGongError那段，也借助于description方便了不同构造的表示问题

{% highlight swift%}
func errorCheck(error:ErrorType?, errorOccured:((err:ErrorType) -> ())? = nil, noError:(()->())? = nil) -> Bool {
    guard let err = error else {
        noError?()
        return true
    }

    switch err {
        case ETeGongError.NeedLogin:
            self.popToast(ETeGongError.NeedLogin.description)
            errorOccured?(err: err)
        case let etErr as ETeGongError:
            self.popToast(etErr.description)
            errorOccured?(err: err)
        case let nsErr as NSError:
            self.popToast(nsErr.localizedDescription)
            errorOccured?(err: err)
        default:()
    }

    return false
}

{% endhighlight %}


对于异步的处理
======

别想着有得throw就觉得简单了，遇到异步的情况还是要老老实实的do-try-catch然后传出去，因为系统不知道要扔去哪里，想起了Xarmain里面那些恶心的提示

于是要定义类似这样的closesure,专门去接受ErrorType
{% highlight swift%}

public func getJSON(basePath: String, interface: String, params: Dictionary<String, AnyObject>, then: (JSON, ErrorType?) -> ())

{% endhighlight %}

