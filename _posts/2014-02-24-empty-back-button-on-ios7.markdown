---
layout: post
title: "Empty Back Button on iOS7"
date: 2014-02-24 21:55:16 +0100
comments: false
categories: [iOS,UIBarButtonItem,BackButton,iOS7,UINavigationBar,UINavigationItem,UINavigationController]
---
On iOS7 the `UINavigationBar` features a nice little back arrow when used together with a `UINavigationItem` and/or `UINavigationController`.

![iOS 7 UINavigationBar]({{ site.baseurl }}/assets/backbutton-default.png)

But what if you wanted the back button text to disappear leaving the back arrow there by itself, turns out it is not as simple as you might expect.

<!--more-->

## Back Button Title Position Adjustment
A popular answer on StackOverflow is simply to call `setBackButtonTitlePositionAdjustment:forBarMetrics:` on `UIBarButtonItem` with a large negative offset e.g. `UIOffsetMake(-1000, -1000)`.

{% highlight objc %}
[[UIBarButtonItem appearance] setBackButtonTitlePositionAdjustment:UIOffsetMake(-1000, -1000) forBarMetrics:UIBarMetricsDefault];
{% endhighlight %}

And at the first glance it actually looked like it works, but as Jelle Vandebeeck points out in [Empty Back Button In iOS 7](http://www.fousa.be/blog/empty-back-button-in-ios-7) the solutions fails when the title of the previous view controller is too large.

![Back button text hidden with UIOffsetMake(-1000, -1000)]({{ site.baseurl }}/assets/backbutton-offset.png)

If we put a breakpoint in `viewDidAppear:` and execute `po [[UIWindow keyWindow] recursiveDescription]` in the console we get the following output.

{% highlight objc %}
<UIWindow: 0x8d6f970; frame = (0 0; 320 480); autoresize = W+H; gestureRecognizers = <NSArray: 0x8d5dbf0>; layer = <UIWindowLayer: 0x8d717d0>>
   | <UILayoutContainerView: 0x8d7bbf0; frame = (0 0; 320 480); autoresize = W+H; gestureRecognizers = <NSArray: 0x8d78a70>; layer = <CALayer: 0x8d7bcd0>>
   |    | <UINavigationTransitionView: 0x8d813f0; frame = (0 0; 320 480); clipsToBounds = YES; autoresize = W+H; layer = <CALayer: 0x8d814d0>>
   |    |    | <UIViewControllerWrapperView: 0x8d61050; frame = (0 0; 320 480); autoresize = W+H; userInteractionEnabled = NO; layer = <CALayer: 0x8d88f40>>
   |    |    |    | <UIView: 0x8ab0dc0; frame = (0 0; 320 480); autoresize = RM+BM; layer = <CALayer: 0x8ab0610>>
   |    |    |    |    | <_UILayoutGuide: 0x8ab0e20; frame = (0 0; 0 64); hidden = YES; layer = <CALayer: 0x8ab0e90>>
   |    |    |    |    | <_UILayoutGuide: 0x8ab1080; frame = (0 480; 0 0); hidden = YES; layer = <CALayer: 0x8ab10f0>>
   |    | <UINavigationBar: 0x8d75c40; frame = (0 20; 320 44); opaque = NO; autoresize = W; userInteractionEnabled = NO; gestureRecognizers = <NSArray: 0x8d5e750>; layer = <CALayer: 0x8d70f00>>
   |    |    | <_UINavigationBarBackground: 0x8d59af0; frame = (0 -20; 320 64); opaque = NO; autoresize = W; userInteractionEnabled = NO; layer = <CALayer: 0x8d549f0>> - (null)
   |    |    |    | <_UIBackdropView: 0x8d7c440; frame = (0 0; 320 64); opaque = NO; autoresize = W+H; userInteractionEnabled = NO; layer = <_UIBackdropViewLayer: 0x8d7e7b0>>
   |    |    |    |    | <_UIBackdropEffectView: 0x8d7f1c0; frame = (0 0; 320 64); clipsToBounds = YES; opaque = NO; autoresize = W+H; userInteractionEnabled = NO; animations = { filters.colorMatrix.inputColorMatrix=<CABasicAnimation: 0x8ba4490>; }; layer = <CABackdropLayer: 0x8d7f480>>
   |    |    |    |    | <UIView: 0x8d7fc80; frame = (0 0; 320 64); hidden = YES; opaque = NO; autoresize = W+H; userInteractionEnabled = NO; layer = <CALayer: 0x8d7fce0>>
   |    |    |    | <UIImageView: 0x8d67cc0; frame = (0 64; 320 0.5); userInteractionEnabled = NO; layer = <CALayer: 0x8d67d50>> - (null)
   |    |    | <UINavigationItemView: 0x8ab6400; frame = (124 8; 163 27); opaque = NO; userInteractionEnabled = NO; layer = <CALayer: 0x8ab6480>>
   |    |    |    | <UILabel: 0x8ab64b0; frame = (0 3; 163 22); text = 'A Story About a Fish'; clipsToBounds = YES; opaque = NO; userInteractionEnabled = NO; layer = <CALayer: 0x8ab6550>>
   |    |    | <UINavigationItemButtonView: 0x8ab6c80; frame = (8 6; 110 30); opaque = NO; userInteractionEnabled = NO; layer = <CALayer: 0x8ab6d60>>
   |    |    |    | <UILabel: 0x8ab6f10; frame = (-981 -995; 91 22); text = 'Passionfruit'; clipsToBounds = YES; opaque = NO; userInteractionEnabled = NO; layer = <CALayer: 0x8ab6fb0>>
   |    |    | <_UINavigationBarBackIndicatorView: 0x8d87560; frame = (8 12; 12.5 20.5); opaque = NO; userInteractionEnabled = NO; layer = <CALayer: 0x8d87650>> - Back
{% endhighlight %}

This reveals that the `UILabel` of the back button indeed has been adjusted by (-1000, -1000) but the superview (`UINavigationItemButtonView`) still has the 'original' frame, which is affecting the frame of the `UINavigationItemView` containg the title label.

## An Empty Back Button
Another approach is to set the `backBarButtonItem` in `viewDidLoad` on the 'parent' view controller to a `UIBarButtonItem` with an empty title, `nil` target and `nil` action as suggested by [Jagie](http://stackoverflow.com/a/21985021/342437).

{% highlight objc %}
- (void)viewDidLoad {
    [super viewDidLoad];
    UIBarButtonItem *backButtonItem = [[UIBarButtonItem alloc] initWithTitle:@"" style:UIBarButtonItemStylePlain target:nil action:nil];
    [self.navigationItem setBackBarButtonItem:backButtonItem];
}
{% endhighlight %}

This approach acutally archives the wanted effect.

![Using an empty back button]({{ site.baseurl }}/assets/backbutton-empty.png)

This is doable if it is only needed in a single view controller, but if this behaviour is application wide a more sustainable and centralized solution is needed.
The option is either subclassing `UIViewController` and then use it as a base class or by method swizzling `viewDidLoad`.

### Subclassing
Subclassing ´UIViewController´ is straight forward and it is something we do all the time when creating apps.
An example of a new base class could be.

{% highlight objc %}
@interface MOBViewController : UIViewController

@end
{% endhighlight %}

{% highlight objc %}
#import "MOBViewController.h"

@implementation MOBViewController

- (void)viewDidLoad
{
    [super viewDidLoad];
    UIBarButtonItem *backButtonItem = [[UIBarButtonItem alloc] initWithTitle:@"" style:UIBarButtonItemStylePlain target:nil action:nil];
    [self.navigationItem setBackBarButtonItem:backButtonItem];
}

@end
{% endhighlight %}

Then every time you create a new view controller just specify `MOBViewController` as the super class.
There is just a small issue with this approach, what if you need to use a `UITableViewController` or a `UICollectionViewController`.
Then you would need similar base classes with the exact same code in them and duplicate code is bad.

### Method swizzling
Since the subclassing approach leads to duplicated code we can use [Method Swizzling](http://nshipster.com/method-swizzling/) to archive the same result without writing the same code twice.

{% highlight objc %}
@interface UIViewController (EmptyBackButton)

@end
{% endhighlight %}

{% highlight objc %}
#import "UIViewController+EmptyBackButton.h"
#import <objc/runtime.h>

@implementation UIViewController (EmptyBackButton)

+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        Class class = [self class];

        SEL originalSelector = @selector(viewDidLoad);
        SEL swizzledSelector = @selector(mob_viewDidLoad);

        Method originalMethod = class_getInstanceMethod(class, originalSelector);
        Method swizzledMethod = class_getInstanceMethod(class, swizzledSelector);

        BOOL didAddMethod = class_addMethod(class, originalSelector, method_getImplementation(swizzledMethod), method_getTypeEncoding(swizzledMethod));

        if (didAddMethod) {
            class_replaceMethod(class, swizzledSelector, method_getImplementation(originalMethod), method_getTypeEncoding(originalMethod));
        } else {
            method_exchangeImplementations(originalMethod, swizzledMethod);
        }
    });
}

#pragma mark - Method Swizzling

- (void)mob_viewDidLoad {
    [self mob_viewDidLoad];
    UIBarButtonItem *backButtonItem = [[UIBarButtonItem alloc] initWithTitle:@"" style:UIBarButtonItemStylePlain target:nil action:nil];
    [self.navigationItem setBackBarButtonItem:backButtonItem];
}

@end
{% endhighlight %}

Now we have specified that we want our back buttons empty in a centralized place with no duplication of code.
