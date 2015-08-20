---
layout: post
title: "Xcode 6 Live Rendering from nib"
date: 2014-07-27 10:09:03 +0200
comments: false
categories: [Xcode,iOS,Interface Builder,Swift,Nib]
permalink: /:year/:month/:day/:title/
---

When [Apple](http://apple.com) announced Xcode 6 at WWDC14, one feature in particular excited me namely **Xcode Live Rendering**.
This means an end to all the empty white views in place for our custom views.

But as Apple states in [What’s new in Xcode 6](https://developer.apple.com/xcode/), it is intended for *hand-written* UI code.

> Live rendering within Interface Builder displays your hand-written UI code within the design canvas, instantly reflecting changes you type in code.

This is unsatisfying for me, since I went all in on Interface Builder when *Storyboards* and *Auto Layout* were introduced.
Also I find it strange that Apple has not made it easier to do custom views using nibs.

<!--more-->

# Nibs

Since I like creating my custom views backed by a nib, I would not let this stop me.
By bending UIKit a little it is possible to get **Live Rendering** working with custom views designed in a nib file.

For this post let us create a very simple view, which displays an image and a name underneath like illustrated below.

![Avatar, a custom UIView]({{ site.baseurl }}/assets/xcode-6-live-rendering-from-nib-example.png)

First let us create `CustomView.swift` and a corresponding nib called `CustomView.xib`.
Add a `UIImageView` and a `UILabel` and connect these to our class.
Lastly add an instance of `CustomView` to our storyboard.

{% highlight swift %}
import UIKit

class CustomView: UIView {
    @IBOutlet weak var avatarImageView: UIImageView!
    @IBOutlet weak var titleLabel: UILabel!

    public var title: String = "" {
        didSet {
            self.titleLabel.text = title
        }
    }

    public var avatarImage: UIImage = UIImage() {
        didSet {
            self.avatarImageView.image = self.avatarImage
        }
    }
}
{% endhighlight %}

Nothing happens yet since Interface Builder does not know anything about our nib file.
So let us take a quick look at how to load a nib file from code.
I am using `NSBundle(forClass:)` instead of `NSBundle.mainBundle()` as it will work if the view is part of the main target or if it is part of a framework.

{% highlight swift %}
let bundle = NSBundle(forClass: self.dynamicType)
var view = bundle.loadNibNamed("CustomView", owner: nil, options: nil)[0] as CustomView
{% endhighlight %}

Now that we can load a nib file, we need to get Interface Builder to use the nib file.
When Interface Builder initializes a view, it calls the following methods in order:

1. `init(coder:)` - Initializer defined in the `NSCoding` protocol.
2. `awakeAfterUsingCoder(aDecoder:)` - Makes it possible to replace the decoded object with another if needed. Default implementation returns `self`.
3. `awakeFromNib()` - Called when every item in the nib have been loaded.

So our only option is to override `awakeAfterUsingCoder(aDecoder:)`, load our view from nib and return it.
We can assume that if our view has 0 subviews, it has not been created from `CustomView.xib`.

{% highlight swift %}
import UIKit

class CustomView: UIView {
    @IBOutlet weak var avatarImageView: UIImageView!
    @IBOutlet weak var titleLabel: UILabel!

    public var title: String = "" {
        didSet {
            self.titleLabel.text = title
        }
    }

    public var avatarImage: UIImage = UIImage() {
        didSet {
            self.avatarImageView.image = self.avatarImage
        }
    }

    override func awakeAfterUsingCoder(aDecoder: NSCoder!) -> AnyObject! {
        if self.subviews.count == 0 {
            let bundle = NSBundle(forClass: self.dynamicType)
            var view = bundle.loadNibNamed("CustomView", owner: nil, options: nil)[0] as CustomView
            view.setTranslatesAutoresizingMaskIntoConstraints(false)
            let contraints = self.constraints()
            self.removeConstraints(contraints)
            view.addConstraints(contraints)
            return view
        }
        return self
    }
}
{% endhighlight %}

Now our custom view is loaded from our nib when the app is executed.
But the whole point is to get a live rendering of our view inside our storyboard.
To do so, we add `@IBDesignable` before our class definition and prepend `@IBInspectable` to the two public properties.

{% highlight swift %}
import UIKit

@IBDesignable
class CustomView: UIView {
    @IBOutlet weak var avatarImageView: UIImageView!
    @IBOutlet weak var titleLabel: UILabel!

    @IBInspectable public var title: String = "" {
        didSet {
            self.titleLabel.text = title
        }
    }

    @IBInspectable public var avatarImage: UIImage = UIImage() {
        didSet {
            self.avatarImageView.image = self.avatarImage
        }
    }

    override func awakeAfterUsingCoder(aDecoder: NSCoder!) -> AnyObject! {
        if self.subviews.count == 0 {
            let bundle = NSBundle(forClass: self.dynamicType)
            var view = bundle.loadNibNamed("CustomView", owner: nil, options: nil)[0] as CustomView
            view.setTranslatesAutoresizingMaskIntoConstraints(false)
            let contraints = self.constraints()
            self.removeConstraints(contraints)
            view.addConstraints(contraints)
            return view
        }
        return self
    }
}
{% endhighlight %}

Switching to our storyboard just shows an empty white box, as we know from the Xcode 5 days.
Strangely **Live Rendering** does not initialize our view using `init(coder:)` but instead uses `init(frame:)` as the entry point when generating the preview.

Due to this behavior, we are forced to load our nib in `init(frame:)` and add it as a subview.

{% highlight swift %}
init(frame: CGRect) {
    super.init(frame: frame)
    let bundle = NSBundle(forClass: self.dynamicType)
    var view = bundle.loadNibNamed("CustomView", owner: nil, options: nil)[0] as CustomView
    view.frame = self.bounds
    view.autoresizingMask = .FlexibleWidth | .FlexibleHeight
    self.addSubview(view)
}
{% endhighlight %}

And we end up with the following view hierarchy:

{% highlight swift %}
CustomView
    CustomView
        UIImageView
        UILabel
{% endhighlight %}

For this to work with our public properties, we need a reference to the loaded `CustomView`

{% highlight swift %}
private var proxyView: CustomView?
{% endhighlight %}

and change the `didSet` observer

{% highlight swift %}
didSet {
    if let optionalView = self.proxyView {
        optionalView.titleLabel.text = title
    }
    else {
        self.titleLabel.text = title
    }
}
{% endhighlight %}

Opening our storyboard, we now see a live rendering of `CustomView` and we can change the inspectable properties.

A success, but the `if let else` pattern in `didSet` does not feel quite right and with a few changes the code can be simplified.
If we in `awakeAfterUsingCoder(aDecoder:)` do `view.proxyView = view` before we return, `didSet` can be changed to.
We can force unwrap `proxyView` since we are certain that it will never be `nil`.

{% highlight swift %}
didSet {
    self.proxyView!.titleLabel.text = title
}
{% endhighlight %}

We now have a custom view backed by a nib that can be used from code and Interface Builder.

{% highlight swift %}
import UIKit

@IBDesignable
class CustomView: UIView {
    @IBOutlet weak var avatarImageView: UIImageView!
    @IBOutlet weak var titleLabel: UILabel!
    private var proxyView: CustomView?

    @IBInspectable public var title: String = "" {
        didSet {
            self.proxyView!.titleLabel.text = title
        }
    }

    @IBInspectable public var avatarImage: UIImage = UIImage() {
        didSet {
            let size = self.avatarImage.size
            let rect = CGRectMake(0, 0, size.width, size.height)
            UIGraphicsBeginImageContextWithOptions(size, false, 0.0)
            var path = UIBezierPath(ovalInRect: rect)
            path.addClip()
            self.avatarImage.drawInRect(rect)

            let image = UIGraphicsGetImageFromCurrentImageContext()
            UIGraphicsEndImageContext()
            self.proxyView!.avatarImageView.image = image
        }
    }

    init(frame: CGRect) {
        super.init(frame: frame)
        var view = self.loadNib()
        view.frame = self.bounds
        view.autoresizingMask = .FlexibleWidth | .FlexibleHeight
        self.proxyView = view
        self.addSubview(self.proxyView)
    }

    init(coder aDecoder: NSCoder!) {
        super.init(coder: aDecoder)
    }

    override func awakeAfterUsingCoder(aDecoder: NSCoder!) -> AnyObject! {
        if self.subviews.count == 0 {
            var view = self.loadNib()
            view.setTranslatesAutoresizingMaskIntoConstraints(false)
            let contraints = self.constraints()
            self.removeConstraints(contraints)
            view.addConstraints(contraints)
            view.proxyView = view
            return view
        }
        return self
    }

    private func loadNib() -> CustomView {
        let bundle = NSBundle(forClass: self.dynamicType)
        var view = bundle.loadNibNamed("CustomView", owner: nil, options: nil)[0] as CustomView
        return view
    }
}
{% endhighlight %}

![Storyboard showing two live rendered views]({{ site.baseurl }}/assets/xcode-6-live-rendering-from-nib-final-storybaord.png)

# Final notes
When working with `CustomView.xib` Interface Builder will naturally live render the view.
This means that any changes are delayed by the rendering time.
So my advice would be to remove `@IBDesignable` when making changes to your nib.

I have created an example project showcasing the above and it can be found on [GitHub](https://github.com/mbogh/xcode-live-rendering).
