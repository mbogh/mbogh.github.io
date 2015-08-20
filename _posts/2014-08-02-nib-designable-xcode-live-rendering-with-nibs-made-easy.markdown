---
layout: post
title: "Nib Designable - Xcode Live Rendering with nibs made easy"
date: 2014-08-02 19:20:52 +0200
comments: false
categories: [Xcode,iOS,Interface Builder,Swift,Nib]
permalink: /:year/:month/:day/:title/
---

In my previous post [Xcode 6 Live Rendering From Nib](http://justabeech.com/2014/07/27/xcode-6-live-rendering-from-nib/), we ended up with `CustomView` a `UIView` subclass backed by a nib that is supported by *Xcode Live Rendering*.

{% highlight swift %}
import UIKit

@IBDesignable
class CustomView: UIView {
    @IBOutlet weak var avatarImageView: UIImageView!
    @IBOutlet weak var titleLabel: UILabel!
    private weak var proxyView: CustomView?

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
        return bundle.loadNibNamed("CustomView", owner: nil, options: nil)[0] as CustomView
    }
}
{% endhighlight %}

This works well for a single custom view, but who only got one custom view, so let me introduce [Nib Designable](https://github.com/mbogh/NibDesignable).

<!-- more -->

Nib Designable is a single file which enables Live Rendering on its subclass'.

{% highlight swift %}
import UIKit

@IBDesignable
public class NibDesignable: UIView {

    // MARK: - Initializer
    override init(frame: CGRect) {
        super.init(frame: frame)
        self.setupNib()
    }

    // MARK: - NSCoding
    required public init(coder aDecoder: NSCoder!) {
        super.init(coder: aDecoder)
        self.setupNib()
    }

    // MARK: - Nib loading

    /**
        Called in init(frame:) and init(aDecoder:) to load the nib and add it as a subview.
    */
    private func setupNib() {
        var view = self.loadNib()
        view.frame = self.bounds
        view.autoresizingMask = .FlexibleWidth | .FlexibleHeight
        self.addSubview(view)
    }

    /**
        Called to load the nib in setupNib().

        :returns: UIView instance loaded from a nib file.
    */
    public func loadNib() -> UIView {
        let bundle = NSBundle(forClass: self.dynamicType)
        return bundle.loadNibNamed(self.nibName(), owner: self, options: nil)[0] as UIView
    }

    /**
        Called in the default implementation of loadNib(). Default is class name.

        :returns: Name of a single view nib file.
    */
    public func nibName() -> String {
        return self.dynamicType.description().componentsSeparatedByString(".").last!
    }
}
{% endhighlight %}

`NibDesignable` is almost identical to `CustomView` from my previous post.
I added `nibName() -> String` to ease the usage.

For more details on Nib Designable, head over to the [Github repository](https://github.com/mbogh/NibDesignable).

# Update
I have updated Nib Designable so it is even easier to use.
No more `proxyView` and you are no longer forced to subclass `nibName()`, go check out the repository over at Github :)
