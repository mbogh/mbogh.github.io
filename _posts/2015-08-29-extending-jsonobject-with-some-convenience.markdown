---
layout: post
title: "Extending JSONObject with some Convenience"
date: 2015-08-29 12:11:10 +0100
comments: false
categories: [swift,ios,JSONObject]
permalink: /extending-jsonobject-with-some-convenience/
---

I have become quite fond of the JSON mapping approach in [No-Magic JSON Parsing with Swift Part 2](http://jasonlarsen.me/2015/06/23/no-magic-json-pt2.html) and extended it with the following convenience method.

{% highlight swift %}
extension JSONObject {
    /// Returns a JSON object initialized by reading into it the data from the file specified by a given path.
    /// - Parameter path: The absolute path of the file from which to read data.
    /// - Returns: A JSON object initialized by reading into it the data from the file specified by path.
    convenience init?(contentsOfFile path: String) {
        guard let data = NSData(contentsOfFile: path),
                  json = (try? NSJSONSerialization.JSONObjectWithData(data, options: .AllowFragments)) as? [String : AnyObject] else { return nil }
        self.init(dictionary: json)
    }
}
{% endhighlight %}

You can also find on GitHub as a [Gist](https://gist.github.com/mbogh/63b3bdc172ef3ce092ac)
