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
- <a href="#adding-static-documents">Adding static documents</a>
- <a href="#limitations">Limitations</a>

### What's Dgeni

Dgeni (pronounced Jenny) is an extremely powerful NodeJS documentation generation utility. It was built by people from the Angular team, so it does make everything easy to document Angular projects although it can be used for other projects as well. The main feature of Dgeni is to convert your source files to documentation files. You can convert them to a full HTML page, partials, or something else.

Dgeni does not provide a web application to display the output files, but it allows you to do what you want with it. It's up to the developer to decide what he wants to do with it. In this example we'll actually wrap our documentation in a simple Angular app.

It has a modular core and offers multiple plugins (packages). You configure your base package using Processors and Services to customize how you want to generate your documentation.

All of your documentation needs to be written in a form of JSDoc, which is a standard for Javascript documentation (although ESDoc is picking up!), and is written inline in the source code. The various Dgeni Processors will then scan and convert your source files to documentation files. This also means that it supports many of the standard JSDoc directives, so if you are already writting JSDoc today, your work won't be lost :)

**Briefly explain Dgeni pipeline and show the site with how it works.**


Key concepts

- Processors: Scan and convert source files, the building blocks of documentation generation
- Services: Helper objects
- Packages: Reusable Component Containers

### Why you should document your code

There are many reasons why you should document your code, but we won't be getting into specifics in this article. The main reason I will evoke here is that it allows your code to be more concise and clear, and this is extremely important when you are working in a team environment.

Working on large scale projects with multiple files, deadlines, employees leaving, code refactorings & rewrites, these things make it hard to keep track.

Having a up-to-date documentation will allow you to:

- Remember the code you wrote 6 months ago
- Easily bring someone new into the project
- Have people understand your code clearly
- Easily understand each other's code
- Easily document business rules and complex algorythms

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

First thing we are going to do is create a `docs` folder where we will have our configuration file, as well as the actual documentation. In this case, I opted to simply put it in the same directory, but you could package it in a `dist` folder if you want.

Open your command line and create the following folders in the root folder of your application
{% highlight shell %}
mkdir docs
cd docs
mkdir config
mkdir content
{% endhighlight %}

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

Using the Gulp task, looking at the partials folder

### How to improve

Wrapping everything in an Angular App

### Adding static documents

Talk about how we can easily add markdown type files and add them to our app

### Limitations
Talk about how Controllers and Components cannot be documented correctly and how we could add our own processors to take care of it

### Conclusion

How do we conclude???
