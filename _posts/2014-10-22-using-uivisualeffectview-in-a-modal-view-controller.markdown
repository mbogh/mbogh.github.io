---
layout: post
title: "Using UIVisualEffectView in a modal view controller"
date: 2014-10-22 19:08:53 +0200
comments: false
categories: [iOS,Swift,Xcode,UIVisualEffectView,UIViewController]
permalink: /:year/:month/:day/:title/
---

If you wanted to spice up your modally presented view controllers for iOS 8 users by adding a `UIVisualEffectView` you will likely end up with the this.

![Awful animation]({{ site.baseurl }}/assets/using-uivisualeffectview-in-a-modal-view-controller-fail.gif)

<!--more-->

{% highlight swift %}
let modalViewController = ModalViewController()
self.presentViewController(modalViewController, animated: true, completion: nil)
{% endhighlight %}

The reason for this is that the view of the presenting view controller is removed from the view stack.
Luckily Apple has us covered with a new `UIModalPresentationStyle` called `UIModalPresentationOverFullScreen`.

> UIModalPresentationOverFullScreen
> A view presentation style in which the presented view covers the screen. The views beneath the presented content are not removed from the view > hierarchy when the presentation finishes. So if the presented view controller does not fill the screen with opaque content, the underlying content shows through.
>
> Available in iOS 8.0 and later.
> [Apple Docs](https://developer.apple.com/library/ios/documentation/uikit/reference/UIViewController_Class/#//apple_ref/c/tdef/UIModalPresentationStyle)

As wanted the views beneath the presented view controller remains on the view stack.
So with almost no effort at all we can get the `UIVisualEffectView` to work as intended on the modal view controller.

{% highlight swift %}
let modalViewController = ModalViewController()
modalViewController.modalPresentationStyle = .OverFullScreen
self.presentViewController(modalViewController, animated: true, completion: nil)
{% endhighlight %}

![All fine here]({{ site.baseurl }}/assets/using-uivisualeffectview-in-a-modal-view-controller-success.gif)

Source code for `ModalViewController` is as follows:

{% highlight swift %}
import UIKit

class ModalViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        self.view.backgroundColor = .clearColor()

        let visuaEffectView = UIVisualEffectView(effect: UIBlurEffect(style: .Light))
        visuaEffectView.frame = self.view.bounds
        visuaEffectView.autoresizingMask = .FlexibleWidth | .FlexibleHeight
        visuaEffectView.setTranslatesAutoresizingMaskIntoConstraints(true)
        self.view.addSubview(visuaEffectView)
    }
}
{% endhighlight %}

Have a lovely day :)  
Morten
