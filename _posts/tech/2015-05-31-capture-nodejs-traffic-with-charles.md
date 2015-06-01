---
layout: post
title: "Capture NodeJS Traffic with Charles"
excerpt: "Simple way to debug your javascript application without any additional dependencies"
modified: 2015-05-31
categories: tech
tags: [nodejs, javascript, charles proxy]
image:
  feature:
comments: true
date: 2015-05-31T00:00:00-00:00
---

Quick and simple approach described below will let your inspect your NodeJS application traffic: both HTTP and SSL / HTTPS. No additional dependencies required: only [Charles proxy][charles].
 
Charles is going to receive all traffic from the application and send to your server.

###### Configuring Charles:
1. Go to *Proxy >> Reverse Proxies...*
2. Add your server as reverse proxy, example: <br>
    `Local Port: 60103` - any unused local port <br>
    `Remote Host: myapi.com` - your host address <br>
    `Remote Port: 443 - port` for your host <br>

* For SSL / HTTPS traffic:

3. Go to *Proxy >> SSL Proxying Settings... >> SSL Proxying* tab
4. Check *Enable SSL Proxying*, add your server, e.g.: <br>
    `Host: myapi.com` <br>
    `Port: 443` <br>

###### Configuring NodeJS application:
1. Edit requests options i.e. point to Charles proxy: <br>
    `host: localhost` <br>
    `port: 60103` - local port from step 2 above <br>
2. Add following line to prevent certificate errors:
{% highlight javascript %}
process.env.NODE_TLS_REJECT_UNAUTHORIZED = "0";
{% endhighlight %}

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

Enjoy debugging!

Charles proxy version used: v 3.10.1. It was tested on Mac OS and should as well works on Linux and Windows. You could also use free [Fiddler proxy][fiddler] on PC.

[charles]:  http://www.charlesproxy.com
[fiddler]:  http://www.telerik.com/fiddler
