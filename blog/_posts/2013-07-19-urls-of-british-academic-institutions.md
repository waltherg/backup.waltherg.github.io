---
layout: post
title: "Domains of British Academic Institutions and Learned Societies"
date: 2013-07-19
categories: uk academia
---

A list of all domains registered by British academic institutions and 
[learned societies](http://en.wikipedia.org/wiki/Learned_society) was
provided by [JANET](http://en.wikipedia.org/wiki/JANET) back in 2011:

[https://www.whatdotheyknow.com/request/list_of_acuk_domain_names]

Since this is 2013 and domain registration probably changes fast we
may want to renew the above freedom of information request.

A JSON version of the data provided by JANET can be found
[here](https://gist.github.com/waltherg/6037738).

Let us play around a little ...

{% highlight js %}
var fs = require('fs');
var json = JSON.parse(fs.readFileSync('ac_uk_domains.json'));
console.log(json['institute_names'].length);
{% endhighlight %}

Running this with nodejs prints the number of institute names covered

{% highlight bash %}
1864
{% endhighlight %}
