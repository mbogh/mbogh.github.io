---
layout: post
title: "NibDesignable available as CocoaPod"
date: 2015-01-13 19:18:45 +0100
comments: false
categories: [swift,ios,xcode,interface builder]
---

With the upcoming version [0.36](http://blog.cocoapods.org/Pod-Authors-Guide-to-CocoaPods-Frameworks/) [CocoaPods](http://cocoapods.org) there is Swift and Framework support.
So I found some time and added a `.podspec` to [NibDesignable](https://github.com/mbogh/NibDesignable).

### Installation

1. Add `pod 'NibDesignable'` to your `Podfile`
2. Add `import NibDesignable`
3. Sometimes Xcode/Interface Builder does not recognize `NibDesignable` as `@IBDesignable`. **Workaround** Declare your custom class as `@IBDesignable` like:

{% highlight swift %}
@IBDesignable
class CustomView: NibDesignable {

}
{% endhighlight %}
