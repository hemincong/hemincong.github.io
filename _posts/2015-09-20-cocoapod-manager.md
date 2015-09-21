---
layout: post
title: Cocoapod 包管理笔记
permalink: /Cocoapod/
---

用CocoaPod进行包管理
======

理由：

1. 不需要自行进行依赖分析、管理以及更新。0.38以后的cocopod还会解决冲突
2. 隔离你自己的代码和别人的代码

说明文档：[cocoapods官网](https://cocoapods.org/)

想看中文的：[唐巧的blog](http://blog.devtang.com/blog/2014/05/25/use-cocoapod-to-manage-ios-lib-dependency) 这个唐巧在业内也是比较有名气的

安装方法 
=====

{% highlight bash%}
$ sudo gem install cocoapods
{% endhighlight %}

必要的时候翻墙，或者换源，看唐巧的blog

使用方法
=====

具体就不说了，看文档。

在有project的目录下初始化:
{% highlight bash%}
$ pod init 
{% endhighlight %}

然后就到打开来改咯,下面是我们的podfile文件：

{% highlight ruby%}
platform :ios, '8.0'
use_frameworks!

target 'native-lib' do
    pod 'SwiftyJSON', '~> 2.2.0'
    pod 'Alamofire', '~> 1.2'
end

workspace 'native-lib'
xcodeproj 'native-lib.xcodeproj'
{% endhighlight %}

其中use_frameworks!是对Swift进行支持指定要加的。

Podfile会放在版本管理里面。将项目clone下来之后，除非需要添加新的依赖，上面的是不需要做的。clone之后只需要进行下面的步骤

{% highlight bash%}
$ cd path/to/project
$ pod install
{% endhighlight %}

第一次 pod会从git clone整个仓库下来，会很慢，耐心等待，可加--verbose看详细进展。还有考虑到github被墙的可能性，科学上网，你懂的。
必要的时候换源，或者加--no-repo-update参数。上面唐巧的blog有写。

不要把Podfile.lock加入ignore

包发布
=====

如果作为一个包进行发布，就需要修改PodSpec文件。
这里native-lib就需要这个文件，让其成为一个私有库，然后native-sample的Podfile对其进行依赖

{% highlight ruby %}
Pod::Spec.new do |s|
  s.name             = "native_lib"
  s.version          = "0.1.0"
  s.summary          = "A short description of native_lib."
  s.description      = <<-DESC
                       An optional longer description of native_lib

                       * Markdown format.
                       * Don't worry about the indent, we strip it!
                       DESC
  s.homepage         = "https://github.com/hemincong/native_lib"
  # s.screenshots     = "www.example.com/screenshots_1", "www.example.com/screenshots_2"
  s.license          = 'MIT'
  s.author           = { "mincong.he" => "mincong.he@gmail.com" }
  s.source           = { :git => "https://github.com/hemincong/native-lib.git", :branch => "develop" }
  # s.social_media_url = 'https://twitter.com/<TWITTER_USERNAME>'

  s.platform     = :ios, '8.0'
  s.requires_arc = true

  s.source_files = "ios/native_lib/native_lib/*.swift"
  #s.resource_bundles = {
    #'native_lib' => ['']
  #}

  # s.public_header_files = 'Pod/Classes/**/*.h'
  # s.frameworks = 'UIKit', 'MapKit'
  # s.dependency 'AFNetworking', '~> 2.3'
  #s.dependency 'Alamofire-SwiftyJSON', :git => "https://github.com/hemincong/Alamofire-SwiftyJSON.git" not allow from specspod
  #s.dependency 'SwiftyJSON'
  s.dependency 'Alamofire'
  #s.dependency 'AwesomeCache'
end
{% endhighlight %}
  
其中，比较重要的是source、source_files、dependency等字段，看例子改吧。注意的是，仓库需要有对应的tag或分支

修改完之后，用

{% highlight bash%}
$ pod lib lint
{% endhighlight %}

进行检查，如果要开源发布，那就越详细越好

对应地，引用库的项目的Podfile内对这个podspec进行引用。其实以后可以考虑发布到其他地方去，甚至push到CocoaPod的库中

{% highlight ruby%}
platform :ios, '8.0'
use_frameworks!

target 'native_sample' do
pod 'native_lib', :podspec => '../../../native-lib/ios/native_lib/native_lib.podspec'
end
{% endhighlight ruby%}

尽量是相对路径
