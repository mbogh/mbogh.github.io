---
layout: post
title: "hosts file productivity tip"
date: 2014-10-13 19:48:10 +0200
comments: false
categories: [Productivity,ghost,hosts,osx,nix]
---

I often find myself revisiting the same sites over and over again when faced with a tedious task.
This is counter productive and is just prolonging the pain :)

So I have found it to be quite useful to simply block these sites locally, whenever I feel they become a problem.

To manage my hosts file I use [Ghost](https://github.com/bjeanes/ghost) which is easily installed from the terminal using `gem install ghost`.
When a site is stealing too much of my time, I simply just block it by typing `sudo ghost add facebook.com`. Ghost will add the entry `127.0.0.1 facebook.com` to `/etc/hosts`.

After 2 or 3 times getting the **Safari Can't Connect to the Server**, you will stop using that site.
