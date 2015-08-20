---
layout: post
title: "ReactiveCocoa: Counting characters in a UITextView"
date: 2014-02-06 19:39:10 +0100
comments: false
categories: [iOS,ReactiveCocoa,FRP,RACChannel]
permalink: /:year/:month/:day/:title/
---

In a pet project I have started working on, I have decided to use [ReactiveCocoa (RAC)](https://github.com/ReactiveCocoa/ReactiveCocoa) together with the pattern [Model View ViewModel (MVVM)](http://en.wikipedia.org/wiki/Model_View_ViewModel).
If you are not familiar with RAC or MVVM for that matter, I can recommend Ash Furrow's book [Functional Reactive Programming on iOS](https://leanpub.com/iosfrp) which is a great book on both topics.

<!--more-->

In my app, the user can create an entity and add a description text with a maximum length of 256 characters. The number of characters remaining is displayed to the user with a label.
This task can fairly easy be done without using RAC like the following.

{% highlight objc %}
#pragma mark - UITextViewDelegate

- (void)textViewDidChange:(UITextView *)textView {
    _charactersRemainingLabel.text = [@(EnityDescriptionMaxLength - (NSInteger)textView.text.length) stringValue];
}
{% endhighlight %}

But this is not much functional or reactive and since I am trying to learn and think functional and reactive I needed to find another way around this.
My first approach was using `rac_signalForSelector:fromProtocol:` to wrap the `UITextViewDelegate` method `textViewDidChange:` like Ash Furrow gave an example of in his book.

{% highlight objc %}
[[self rac_signalForSelector:@selector(textViewDidChange:) fromProtocol:@protocol(UITextViewDelegate)] subscribeNext:^(RACTuple *arguments) {
    UITextView *textView = arguments.first;
    _charactersRemainingLabel.text = [@(EnityDescriptionMaxLength - (NSInteger)textView.text.length) stringValue];
}];
{% endhighlight %}

So each time the UITextView calls `textViewDidChange:` on its delegate the signal fires and I get notified in the `subscribeNext` block.
This certainly looks better and the code can be place near the `_charactersRemainingLabel`.

Then the other day, I stumbled upon [RACChannel](http://cocoadocs.org/docsets/ReactiveCocoa/2.1.3/Classes/RACChannel.html) which made me think I could do better.
After a little reserach ([#1069](https://github.com/ReactiveCocoa/ReactiveCocoa/issues/1069)) I ended up with the following code.

{% highlight objc %}
RACChannelTerminal *characterRemainingTerminal = RACChannelTo(_charactersRemainingLabel, text);
[[_textView.rac_textSignal map:^id(NSString *text) {
    return [@(EnityDescriptionMaxLength - (NSInteger)text.length) stringValue];
}] subscribe:characterRemainingTerminal];
{% endhighlight %}

A `RACChannelTerminal` is created for `_charactersRemainingLabel` and the property `text`, which is subscribed to the `rac_textSignal` signal on `UITextView`. So each time the text changes the `rac_textSignal` fires and the value is first mapped and then passed to the channel terminal.

![RACChannelTo in action]({{ site.baseurl }}/assets/RACChannelTo.gif)

I find the final form kinda beautiful :)

Catch me on Twitter [@mbogh](http://twitter.com/mbogh) if you have any comments or want to discuss.
