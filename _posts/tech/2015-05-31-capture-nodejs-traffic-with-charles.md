---
layout: post
title: "Capture NodeJS Traffic with Charles Proxy"
excerpt: "Simple way to inspect traffic of NodeJS application"
modified: 2015-05-31
categories: tech
tags: [nodejs, javascript, charles proxy]
image:
  feature:
comments: true
date: 2015-05-31T00:00:00-00:00
---

Inspecting traffic is very valuable for debugging and troubleshooting. 
Let's assume that your application calls to `myapi.com` and receives unexpected data. Want to view actual request and/or response headers and body? [Charles proxy][charles] can receive traffic from the application and send it to the target server: `myapi.com` while capturing all information.

Quick and simple approach described below will let capture your NodeJS application traffic: both HTTP and SSL / HTTPS. No additional dependencies required: only [Charles proxy][charles].


###### Configuring Charles:
1. Go to *Proxy >> Reverse Proxies...*
2. Check *Enable Reverse Proxies*, add a target server: <br>
    `Local Port: 60103` - any unused local port <br>
    `Remote Host: myapi.com` - target host address <br>
    `Remote Port: 443` - target host port <br>

* For SSL / HTTPS traffic:

3. Go to *Proxy >> SSL Proxying Settings... >> SSL Proxying* tab
4. Check *Enable SSL Proxying*, add a target server: <br>
    `Host: myapi.com` <br>
    `Port: 443` <br>

###### Configuring NodeJS application:
1. Change requests options to point to Charles proxy instead of `myapi.com`: <br>
    `host: localhost` <br>
    `port: 60103` - local port from reverse proxy settings <br>
2. Charles uses self-generated SSL certificates, so to prevent certificate errors add following line:
{% highlight javascript %}
process.env.NODE_TLS_REJECT_UNAUTHORIZED = "0";
{% endhighlight %}

Done! Enjoy debugging.

You could apply the rules only for debugging or development environment, for example with environment variable.

{% highlight javascript %}
var HTTPS = require('https');

// Default options
var options = {
    host: 'myapi.com',
    path: '/',
    method: 'POST',
    protocol: 'https:',
    port: '443'
};

// Edit options if debugging
if (process.env.NODE_ENV === "development") {
    process.env.NODE_TLS_REJECT_UNAUTHORIZED = "0";
    options.host = 'localhost';
    options.port = '60103';
}

var request = HTTPS.request(options, function(res) {
    /* process */
});
{% endhighlight %}

Charles proxy version used: v 3.10.1. It was tested on Mac OS and should work on Linux and Windows as well. You could also use free [Fiddler proxy][fiddler] on PC.

[charles]:  http://www.charlesproxy.com
[fiddler]:  http://www.telerik.com/fiddler
