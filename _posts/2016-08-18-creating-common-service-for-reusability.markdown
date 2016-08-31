---
layout: post
title:  "AngularJS - Creating a common .service() for re-usability"
date:   2016-08-18 09:15:04 -0400
permalink: /javascript/creating-common-service-for-reusability/
category: javascript
---

AngularJS has a neat feature that is dependency injection. It is really easy to inject Services in your Controller so that you can interact with other controllers and it encourages re-usability. But, once your application increases in size and scope, the injectables list gets a little longer and harder to maintain.

Well, there's a solution for that, and it's creating a 'common' Service so that we can inject our most-used dependencies, and not have to manage a big list of dependencies.

# Preparation

# Creating the service

# Using the Service in controllers

# Conclusion

{% highlight javascript %}
angular
    .module('app')
    .service('myService', MyService);

    function MyService() {
        console.log('yo');
    }
{% endhighlight %}
