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

The main feature of dgeni is to convert source code to some kind of document file. This could be a full HTML page, a partial HTML page, a markdown encoded text file or something else. Dgeni does not provide the wrapper web application to display these output files. That is up to the individual developer. That being said the long term aim is to provide a kind of container web app that could accept a standardized set of files output from dgeni.

Dgeni is an extremely powerful documentation generator framework. It's built by people from the Angular team, particularly suitable for Angular projects but I've used it successfully for non-Angular projects as well.

We use the following Dgeni packages:

    NgDoc: Generates API references for anything that is part of JSDoc, with added support for directives, controllers, filters, services and providers.
    Examples: Generates runnable examples. We use this to provide each component with a demonstration of its features, along with code to get people started with using the component.

In addition, we created a custom "Schemas" package to document JSON contracts, component parameters and their connections. The package contains:

    A file reader: Reads out TypeScript files and adds all the defined interfaces to the documentation index (using doc type "schema").
    A processor: Parses the "schema" docs through the Typson JSON Schema generator.
    A rendering filter: Connects documented @params (from directives and functions) with a schema doc if there is one with a matching name.

The Dgeni framework resembles Angular in a lot of ways. The package interface, using Angular's injector on Node.js (of course), makes it very easy to create your own documentation generator based on your specific needs.

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
- Canonical Path (optional), desc      the module is used to generate absolute paths for your source files:
- LoDash (optional), desc

We can easily install all of those dependencies using NPM:
{% highlight shell %}
npm i dgeni dgeni-packages canonical-path lodash --save-dev
{% endhighlight %}

** Talk about using gulp... and make a comment about cleaning up the docs folder because Dgeni doesn't remove the partials (no cleaning) **

{% highlight javascript %}
// Canonical path provides a consistent path (i.e. always forward slashes) across different OSes
var path = require('canonical-path');

var Package = require('dgeni').Package;
var packagePath = __dirname;

module.exports = new Package('cma-docs', [
    require('dgeni-packages/jsdoc'),
    require('dgeni-packages/ngdoc'),
    require('dgeni-packages/nunjucks')
])


.config(function(dgeni, log, readFilesProcessor, templateFinder, writeFilesProcessor) {

    // Set the log level to 'info', switch to 'debug' when troubleshooting
    log.level = 'info';

    // Specify the base path used when resolving relative paths to source and output files
    readFilesProcessor.basePath = path.resolve(packagePath, '..');

    // Specify collections of source files that should contain the documentation to extract
    readFilesProcessor.sourceFiles = [
        { include: '../src/app/common/app.module.js', basePath: 'src' },
        { include: 'content/**/*.ngdoc', basePath: 'content' }
    ];

    // Use the writeFilesProcessor to specify the output folder for the extracted files
    writeFilesProcessor.outputFolder = 'pages';

    // Specify where the templates are located
    templateFinder.templateFolders.unshift(path.resolve(packagePath, 'templates'));

})

.config(function(computePathsProcessor, computeIdsProcessor) {

    computePathsProcessor.pathTemplates.push({
        docTypes: ['module'],
        getPath: function(doc) { return doc.area + '/' + doc.name; },
        outputPathTemplate: 'partials/${path}.html'
    });

    computePathsProcessor.pathTemplates.push({
        docTypes: ['overview'],
        getPath: function(doc) {
            var docPath = path.dirname(doc.fileInfo.relativePath);
            if (doc.fileInfo.baseName !== 'index') {
                docPath = path.join(docPath, doc.fileInfo.baseName);
            } else {
                return 'index';
            }
            return docPath;
        },
        outputPathTemplate: 'partials/${path}.html'
    });

    computePathsProcessor.pathTemplates.push({
        docTypes: ['componentGroup'],
        pathTemplate: '${area}/${moduleName}/${groupType}',
        outputPathTemplate: 'partials/${area}/${moduleName}/${groupType}.html'
    });

    computeIdsProcessor.idTemplates.push({
        docTypes: ['overview'],
        getId: function(doc) { return doc.fileInfo.baseName; },
        getAliases: function(doc) { return [doc.id]; }
    });

})
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
