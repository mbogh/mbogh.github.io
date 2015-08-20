---
layout: post
title: "prepareForInterfaceBuilder and property observers"
date: 2014-08-03 21:56:00 +0200
comments: false
categories: [Xcode,iOS,Interface Builder,Swift]
---

If you are using the `willSet` and `didSet` property observers in combination with `@IBInspectable` and `prepareForInterfaceBuilder()` please notice that Xcode does things in this order:

1. Calls `init(frame:)` on your class.
2. Sets all the `@IBInspectable` properties with the values from Interface Builder
3. Calls `prepareForInterfaceBuilder()` where you might take advantage of your `@IBInspectable` properties again.

This leads to confusion since the values you change in Interface Builder does not take effect as they are overriden by `prepareForInterfaceBuilder()`.

A way around this, is to check whether the property has its *default* value or not:

{% highlight swift %}
import UIKit

@IBDesignable
class MyView: UIView {
    @IBOutlet weak var nameLabel: UILabel!

    @IBInspectable public var name: String = "" {
        didSet {
            self.nameLabel.text = name
        }
    }

    override func prepareForInterfaceBuilder() {
        if countElements(self.name) == 0 {
            self.name = "John Appleseed"
        }
    }
}
{% endhighlight %}

A bug report is in the making for Apple, will update this post when/if a response is posted.

And now back to Objective-C since my vacation has ended :)
