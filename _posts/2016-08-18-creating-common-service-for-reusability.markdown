---
layout: post
title:  "AngularJS - Creating a common .service() for re-usability"
date:   2016-08-18 09:15:04 -0400
permalink: /javascript/creating-common-service-for-reusability/
category: javascript
---

This is a very simple post

{% highlight javascript %}
angular
    .module('app')
    .service('myService', MyService);

    function MyService() {
        console.log('yo');
    }
{% endhighlight %}
