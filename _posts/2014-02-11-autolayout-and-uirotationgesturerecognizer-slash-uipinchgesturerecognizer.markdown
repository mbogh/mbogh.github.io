---
layout: post
title: "Auto Layout and UIRotationGestureRecognizer + UIPinchGestureRecognizer"
date: 2014-02-11 11:51:12 +0100
comments: false
categories: [Auto Layout,UIRotationGestureRecognizer,UIPinchGestureRecognizer,iOS,Xcode,Storyboard,Nib,Xib]
permalink: /:year/:month/:day/:title/
---

On a new project I recently joined, I was given the task to implement a feature which involved panning, zooming and rotating a `UIImageView`.
A straightforward task and something I have done a few times before, so I went and wired up everything in the storyboard and found some old code for handling the three required actions. So far so good.

<!--more-->

{% highlight objc %}
#pragma mark - Gesture recognizer actions

- (IBAction)didPan:(UIPanGestureRecognizer *)panGestureRecognizer {
    CGPoint translation = [panGestureRecognizer translationInView:_bearImageView.superview];
    _bearImageView.center = CGPointMake(_bearImageView.center.x + translation.x, _bearImageView.center.y + translation.y);
    [panGestureRecognizer setTranslation:CGPointZero inView:_bearImageView.superview];
}

- (IBAction)didPinch:(UIPinchGestureRecognizer *)pinchGestureRecognizer {
    _bearImageView.transform = CGAffineTransformScale(_bearImageView.transform, pinchGestureRecognizer.scale, pinchGestureRecognizer.scale);
    pinchGestureRecognizer.scale = 1;
}

- (IBAction)didRotate:(UIRotationGestureRecognizer *)rotationGestureRecognizer {
    _bearImageView.transform = CGAffineTransformRotate(_bearImageView.transform, rotationGestureRecognizer.rotation);
    rotationGestureRecognizer.rotation = 0;
}
{% endhighlight %}

The reason that I am using `_bearImageView` and not `panGestureRecognizer.view` is that all three recognizers are added to the `superView` to give a much better user experience as explained by Ole Begemann in [Gesture Recognition on iOS with Attention to Detail](http://oleb.net/blog/2012/01/gesture-recognition-on-ios-with-attention-to-detail/).

A few seconds later the app was running on my device, but something was not quite right. The whole experience was odd because the image would not overflow the edges of the `superView`, rotation was done around what seemed an arbitrary point, and any panning action would be reset by subsequent rotate or scale gestures.
![Pan, rotate and zoom bugging]({{ site.baseurl }}/assets/buggypanrotatezoom.gif)

I figured that the culpit might be **Auto Layout** since it was the only new thing in town and the code had previously worked.
After some tests and crawling StackOverflow, the conclusion seemed to be adding the `UIImageView` programatically since **Interface Builder** apparently is adding hidden constraints. These contraints would then interfer with the transformations of the `UIImageView`.
![Pan, rotate and zoom working]({{ site.baseurl }}/assets/panrotatezoom.gif)
This looks and feels much better and I have put together a small sample project on GitHub called [RotatingAutoLayout](https://github.com/mbogh/RotatingAutoLayout). Credit goes to [Juliejean Bell](http://www.flickr.com/photos/wildlife2wildplaces777/) for the image of the beautiful bear :)

Success!
