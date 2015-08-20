---
layout: post
title: "Swift makes the little things more simple"
date: 2014-06-04 22:06:12 +0200
comments: false
categories: [iOS,WWDC,Swift,Objective-C]
permalink: /:year/:month/:day/:title/
---
On monday 2014-06-02 Apple introduced a new scripting-style programming language called [Swift](https://developer.apple.com/swift/). As an iOS developer I find this extremely interesting and looking forward to learn this new language and hopefully boost my productivity :)

Swift introduces a lot of nitfy new functions which makes your life as a developer much easier and hides away the boring details. Here is a showcase of `map`, `reduce` and `filter`.

## Map

{% highlight objc %}
NSArray *names = @[@"John", @"Steve", @"Tim"];
NSMutableArray *namesTemporary = [NSMutableArray new];
names enumerateObjectsUsingBlock:^(NSString *name, NSUInteger idx, BOOL *stop) {
    [namesTemporary addObject:name.lowercaseString];
}];
names = [namesTemporary copy];
{% endhighlight %}

{% highlight swift %}
var names: String[] = ["John", "Steve", "Tim"]
names = names.map{ name in name.lowercaseString }
{% endhighlight %}

{% highlight swift %}
names = names.map{ $0.lowercaseString }
{% endhighlight %}

<!--more-->

## Reduce

{% highlight objc %}
NSArray *numbers = @[@1, @2, @3, @4, @5, @6, @7, @8, @9, @10];
__block NSUInteger number = 0;
[numbers enumerateObjectsUsingBlock:^(NSNumber *num, NSUInteger idx, BOOL *stop) {
    number += num.integerValue;
}];
{% endhighlight %}

{% highlight swift %}
let numbers: Int[] = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
let number = numbers.reduce(0){ previous, next in previous + next }
{% endhighlight %}

{% highlight swift %}
let number = numbers.reduce(0){ $0 + $1 }
{% endhighlight %}

## Filter
{% highlight objc %}
NSArray *moreNumbers = @[@1, @2, @3, @4, @5, @6, @7, @8, @9, @10];
moreNumbers = [moreNumbers filteredArrayUsingPredicate:[NSPredicate predicateWithFormat:@"modulus:by:(SELF, 2) = 0"]];
{% endhighlight %}

{% highlight swift %}
var moreNumbers: Int[] = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
moreNumbers = moreNumbers.filter{ number in number % 2 == 0 }
{% endhighlight %}

{% highlight swift %}
moreNumbers = moreNumbers.filter{ $0 % 2 == 0 }
{% endhighlight %}

With `map` and `reduce` the Swift code is much more simple than the Objective-C counter part. With `filter` we can take advantage of the powerful `NSPredicate` to do the magic.
I know that I will be using the `map` function a lot as much of what we do is transforming values into other values.
