---
layout: post
title: "Hardware IO Tools for Xcode"
date: 2015-01-12 13:37:05 +0100
comments: false
categories: [Xcode,Tools,iOS,OS X]
---

I have recently discovered that if you place the applications included in the [Hardware IO Tools for Xcode](https://developer.apple.com/downloads/index.action?name=for Xcode -) bundle inside `/Applications/Xcode.app/Contents/Applications/`.
They will be available through `Xcode -> Open Developer Tools`.

<!--more-->

![All the hardware tools!]({{ site.baseurl }}/assets/hardware-io-tools-for-xcode-open-developer-tools.png)

Now they have a permanent location to live and not inside a `dmg` located in `~/Downloads/`.
