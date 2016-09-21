---
layout: post
permalink: /documenting-angular-dgeni
title: Documenting your Angular app using Dgeni
path: 2016-09-20-document-angular-dgeni.md
---

Having worked on Enterprise-grade solutions, documentation is always an issue. It's either outdated, deprecated or worse, there's no documentation! Generating documentation of your Angular application is easy using Dgeni, which is ironically not well documented for a documentation generator.

### Table of contents

- <a href="#whats-dgeni">What's Dgeni?</a>
- <a href="#why-you-should-document-your-code">Why you should document your code</a>
- <a href="#installing-and-configuration">Installing and Configuration</a>
- <a href="#documenting-your-code">Documenting your code</a>
- <a href="#generating-documentation">Generating documentation</a>
- <a href="#how-to-improve">How to improve</a>
- <a href="#adding-other-parts">Adding other parts</a>

### What's Dgeni

Dgeni (pronounced Jenny) is a NodeJS documentation generation utility and it was created by the AngularJS team. The ideas behind Dgeni were to create a flexible and extensible documentation generator that would allow anyone to document their codebase.

It has a modular core and offers multiple plugins (packages). You configure your base package using Processors and Services to customize how you want to generate your documentation.

All of your documentation needs to be written in a form of JSDoc, which is a standard for Javascript documentation (although ESDoc is pickup up!), and is written inline in the source code. The various Dgeni Processors will then scan and convert your source files to documentation files. This also means that it supports many of the standard JSDoc directives, so if you are already writting JSDoc today, your work won't be lost :)

**Briefly explain Dgeni pipeline and show the site with how it works.**

Key concepts
- Processors: Building blocks of documentation generation
- Services: Singleton Helper Objects
- Packages: Reusable Component Containers

### Why you should document your code

The main things here are `Promise` and the `resolve` and `reject` arguments:

{% highlight javascript %}
var js = new Date();
{% endhighlight %}

We simply call `new Promise()` and inside can perform an asynchronous task, which may be for wrapping particular DOM events, or even wrapping third-party libraries that are not promise Objects.

For example, wrapping a pseudo third-party library called `myCallbackLib()` which gives us a success and error callback, we can construct a Promise around this to `resolve` and `reject` where appropriate:

{% highlight javascript %}
return text;
{% endhighlight %}

### Installing and Configuration

We will need a couple of NodeJS packages for documenting our application:
- Dgeni, desc
- Dgeni Packages, desc
- Canonical Path (optional), desc
- LoDash (optional), desc

We can easily install all of those dependencies using NPM:
{% highlight shell %}
npm i dgeni dgeni-packages canonical-path lodash --save-dev
{% endhighlight %}

### Documenting your code

In the interest of time, I will actually be documenting an existing Angular 1.5 app.

I'm using Todd Motto's (the owner of this blog) Angular 1.5 Component app as a living example. If you haven't had the time to check it out, do so... it's the best example of a .component() based application using UI-Router 1.0 beta using routed components.

**LINK TO APP**

### Generating Documentation

Using `$q.defer()` is just another flavour, and the original implementation, of `$q()` as a Promise constructor. Let's assume the following code, adapted from the before example using a service:

{% highlight javascript %}
return;
{% endhighlight %}

### How to improve

Use `$q.when()` or `$q.resolve()` (they are identical, `$q.resolve()` is an alias for `$q.when()` to align with ES2015 Promise naming conventions) when you want to immediately resolve a promise from a non-promise Object, for example:

{% highlight javascript %}
return;
{% endhighlight %}

Note: `$q.when()` is also the same as `$q.resolve()`.

### Adding other parts

Using `$q.reject()` will immediately reject a promise, this comes in handy for things such as HTTP Interceptors at the point of no return to return, so we can just return a rejected promise Object:

{% highlight javascript %}
return;
{% endhighlight %}

### Conclusion

Use `$q` for constructing promises from non-promise Objects/callbacks, and utilise `$q.all()` and `$q.race()` to work with existing promises.

If you like this article, check out my advanced [Angular 1.5 master course](https://courses.toddmotto.com/products/ultimate-angularjs-master) which covers all the `$q`, `$httpProvider.interceptors`, `ui-router`, component architecture and the new `.component()` API.

For anything else, [the $q documentation](https://docs.angularjs.org/api/ng/service/$q).
