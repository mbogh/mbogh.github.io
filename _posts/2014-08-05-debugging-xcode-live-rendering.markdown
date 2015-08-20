---
layout: post
title: "Debugging Xcode Live Rendering"
date: 2014-08-05 15:52:57 +0200
comments: false
categories: [Xcode,iOS,Interface Builder,Swift,Nib,Debug]
permalink: /:year/:month/:day/:title/
---

If you have experimented with Xcode Live Rendering, you have most likely seen errors like these:

{% highlight text %}
Failed to update auto layout status: Interface Builder Cocoa Touch Tool crashed
{% endhighlight %}

{% highlight text %}
Failed to render instance of ProfileAvatarView: The designables agent crashed
{% endhighlight %}

Placing `breakpoints` in `prepareForInterfaceBuilder()` will for obvious reasons never be hit.
So it is frustrating to get these error messages, when you can not get a better explaination than something crashed.

<!--more-->

Since `breakpoints` are out of the question and there does not seem to be an accessible log file or console for the internals of Xcode/Interface Builder which could provide insight, I have been using plain old log files.

I have created an extension to my class `NibDesignable` but it could just as well be an extension of `UIView`.

{% highlight swift %}
import Foundation

extension UIView {
    public func liveDebugLog(message: String) {
        #if !(TARGET_OS_IPHONE)
            let logPath = "/tmp/XcodeLiveRendering.log"
            if !NSFileManager.defaultManager().fileExistsAtPath(logPath) {
                NSFileManager.defaultManager().createFileAtPath(logPath, contents: NSData(), attributes: nil)
            }

            var fileHandle = NSFileHandle(forWritingAtPath: logPath)
            fileHandle.seekToEndOfFile()

            let date = NSDate()
            let bundle = NSBundle(forClass: self.dynamicType)
            let application: AnyObject = bundle.objectForInfoDictionaryKey("CFBundleName")
            let data = "\(date) \(application) \(message)\n".dataUsingEncoding(NSUTF8StringEncoding, allowLossyConversion: true)
            fileHandle.writeData(data)
        #endif
    }
}
{% endhighlight %}

Then add `liveDebugLog(message:)` where ever you want to gain insight, run `open /tmp/XcodeLiveRendering.log` from `Terminal.app` and watch in `Console.app` as Xcode live renders your view.

{% highlight swift %}
2014-08-05 18:29:40 +0000 NibDesignableDemo init(frame:)
2014-08-05 18:29:40 +0000 NibDesignableDemo didSet:name -> John
2014-08-05 18:29:40 +0000 NibDesignableDemo prepareForInterfaceBuilder
2014-08-05 18:29:40 +0000 NibDesignableDemo didSet:name -> John Appleseed
{% endhighlight %}

If you know of a better way of debugging Xcode Live Rendering, please let me know [@mbogh](https://twitter.com/mbogh)
