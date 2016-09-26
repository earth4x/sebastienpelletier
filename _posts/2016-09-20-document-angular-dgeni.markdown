---
layout: post
permalink: /documenting-angular-dgeni
title: Documenting your Angular app using Dgeni
path: 2016-09-20-document-angular-dgeni.md
---

Having worked on Enterprise-grade solutions, documentation is always an issue. It's either outdated, deprecated or worse, there's no documentation! Generating documentation of your Angular application is easy using Dgeni, which is ironically not well documented for a documentation generator.

In this example I will show you how you can easily document your AngularJS application using Dgeni, and then output that documentation into a separate application!

### Table of contents

- <a href="#whats-dgeni">What's Dgeni?</a>
- <a href="#why-you-should-document-your-code">Why you should document your code</a>
- <a href="#installing-npm-dependencies">1. Installing NPM dependencies</a>
- <a href="#create-folder-structure">2. Create folder structure</a>
- <a href="#setup-configuration-file">3. Setup configuration file</a>
- <a href="#creating-static-documentation">4. Creating static documentation</a>
- <a href="#basic-export-to-html-partials">5. Basic export to HTML partials</a>
- <a href="#creating-an-angular-app-to-show-the-documentation">6. Creating an Angular app to show the documentation</a>
- <a href="#creating-a-processor-for-our-angular-index-page">7. Creating a processor for our Angular index page</a>
- <a href="#generating-a-list-of-pages-for-our-sidebar">8. Generating a list of pages for our sidebar</a>
- <a href="#documenting-a-module-controller-directive-and-service">9. Documenting a Module, Controller, Directive and Service</a>
- <a href="#compile-and-deploy">10. Compile and deploy</a>
- <a href="#limitations">Limitations</a>


### What's Dgeni

Dgeni (pronounced Jenny) is an extremely powerful NodeJS documentation generation utility. It was built by people from the Angular team, so it does make everything easy to document Angular projects although it can be used for other projects as well. AngularJS, Ionic Framework, Protractor and Angular Material are currently using Dgeni in production to generate their documentation.

The main feature of Dgeni is to convert your source files to documentation files. You can convert them to a full HTML page, partials, or something else. It has a modular core and offers multiple plugins (packages). You configure your base package using Processors and Services to customize how you want to generate your documentation.

All of your documentation needs to be written in a form of JSDoc, which is a standard for Javascript documentation (although ESDoc is picking up!), and is written inline in the source code. The various Dgeni Processors will then scan and convert your source files to documentation files. This also means that it supports many of the standard JSDoc directives, so if you are already writting JSDoc today, your work won't be lost :)

Dgeni does not provide a web application to display the output files, but it allows you to do what you want with it. It's up to the developer to decide and n this example we'll actually wrap our documentation in a simple Angular app.


### Why you should document your code

There are many reasons why you should document your code, but we won't be getting into specifics in this article. The main reason I will evoke here is that it allows your code to be more concise and clear, and this is extremely important when you are working in a team environment.

Working on large scale projects with multiple files, deadlines, employees leaving, code refactorings & rewrites, these things make it hard to keep track.

Having a up-to-date documentation will allow you to:

- Remember the code you wrote 6 months ago
- Easily bring someone new into the project
- Easily understand each other's code more clearly
- Document business rules and complex algorythms
- Document dependencies between modules and services

### 1. Installing NPM dependencies

We will need a couple of NodeJS packages for documenting our application:
- Dgeni, our documentation generator
- Dgeni Packages, which are a collection of dgeni packages for generating documentation from source code
- Canonical Path (optional), used to generate absolute paths for your source files
- LoDash (optional), JavaScript utility library

We can easily install all of those dependencies using NPM:

````javascript
npm i dgeni dgeni-packages canonical-path lodash --save-dev
````

You will also need to use Gulp or Grunt to run Dgeni, so I suggest you install it. In my case, it's already in use in my project so it doesn't require any installation. Please note that you can also run Dgeni directly in Node, you would only need to create a script for the generation.

If you are not using Gulp or Grunt in your project, install it globally as this will allow you to follow along and generate the documentation

````javascript
npm i gulp -g
````

### 2. Create folder structure

First thing we are going to do is create a `docs` folder where we will have our configuration files, our static content as well as the actual documentation. In this case, I opted to simply put it in the same directory under `build`, but you could package it in a `dist` folder if you want.

Create the following folder structure in the root folder of your application

````javascript
├── docs/
│   ├── app/
│   ├── config/
│   │  ├── processors/
│   │  ├── templates/
│   │  ├── index.js
│   ├── content/
````

Under `config`, we will be creating our configuration file (index.js) and we will also be adding our Processors and Templates. In the future you can also add Services and Tag Definitions (to parse your own custom tags)

Under `content`, we will be adding our static documentation. Dgeni reads `.md` files by default (using the ngDocFileReader) and will convert them to HTML partials once it's setup correctly.

### 3. Setup configuration file
Now, let's open up `/docs/config/index.js`, and let's start configuring this beast !

````javascript
var path = require('canonical-path');
var packagePath = __dirname;

var Package = require('dgeni').Package;

// Create and export a new Dgeni package
// We will use Gulp later on to generate that package
// Think of packages as containers, our 'myDoc' package contains other packages
// which themselves include processors, services, templates...
module.exports = new Package('myDoc', [
    require('dgeni-packages/ngdoc'),
    require('dgeni-packages/nunjucks')
])
````

Alright, so we've loaded Dgeni, our dgeni packages dependencies and created a new package for us to generate documentation. Next step, we will tell Dgeni which files we want to process and where to output them

````javascript
.config(function(log, readFilesProcessor, writeFilesProcessor) {

    // Set the log level to 'info', switch to 'debug' when troubleshooting
    log.level = 'info';

    // Specify the base path used when resolving relative paths to source and output files
    readFilesProcessor.basePath = path.resolve(packagePath, '../..');

    // Specify our source files that we want to extract
    readFilesProcessor.sourceFiles = [
        { include: 'src/app/**/**/*.js', basePath: 'src/app' }, // All of our application files
    ];

    // Use the writeFilesProcessor to specify the output folder for the extracted files
    writeFilesProcessor.outputFolder = 'docs/build';

})
````

Next, let's specify where are custom templates are located

````javascript
.config(function(templateFinder) {
    // Specify where the templates are located
    templateFinder.templateFolders.unshift(path.resolve(packagePath, 'templates'));
})
````

Looks pretty simple so far? We are merely using the default Dgeni processors to specify how we want to process and then convert our source files. Next, let's setup how we want to convert the source files for each document type

````javascript
.config(function(computePathsProcessor) {

    // Here we are defining what to output for our docType Module
    //
    // Each angular module will be extracted to it's own partial
    // and will act as a container for the various Components, Controllers, Services in that Module
    // We are basically specifying where we want the output files to be located
    computePathsProcessor.pathTemplates.push({
        docTypes: ['module'],
        pathTemplate: '${area}/${name}',
        outputPathTemplate: 'partials/${area}/${name}.html'
    });

    // Doing the same thing but for regular types like Services, Controllers, etc...
    // By default they are grouped in a componentGroup and processed
    // via the generateComponentGroupsProcessor internally in Dgeni
    computePathsProcessor.pathTemplates.push({
        docTypes: ['componentGroup'],
        pathTemplate: '${area}/${moduleName}/${groupType}',
        outputPathTemplate: 'partials/${area}/${moduleName}/${groupType}.html'
    });

})
````

And... that's it! Although Dgeni as been lightly configured, if you we're to run a Gulp/Grunt task, you would immediately see something like this:

![initial gulp task](img/dgeni/gulp-firstParse.png)

Even though we haven't actually started documenting anything, we can see the Dgeni pipeline and how it's going through each different type of processors and performing various actions.


### 4. Creating static documentation

You may be wondering, Why would I want to do static documentation? Well, you may want to document other things besides the code. You may have a Developer guide, REST API References, Tutorials, Notes, Information regarding Unit & E2E tests, libraries behind used in the project... basically anything you can think of!

Once you load the `dgeni-packages` module Dgeni is able to parse `@ngdoc` tags. We can actually use the `overview` tag from ngDoc to create our static documentation, but wouldn't it be nice if we kept the tag as-is and just created a new doctype for Dgeni to parse? That's exactly what we are going to do!

In this example we'll add a simple developer guide, just to show you how easy it is to had static documentation.

Let's start out by creating these Markdown files

````javascript
├── docs/
│   ├── app/
│   ├── config/
│   ├── content/
│   │  ├── api/
│   │  │   ├── index.md
│   │  ├── guide/
│   │  │   ├── index.md
│   │  │   ├── howTo.md
````

Now that we've created those placeholders files and folders, let's write some content! Open up `guide/howTo.md` and type the following

````markdown
@ngdoc content
@name How to write documentation
@description

# Lorem Ipsum
Aenean ornare odio elit, eget facilisis ipsum molestie ac. Nam bibendum a nibh ut ullamcorper.
Donec non felis gravida, rutrum ante mattis, sagittis urna. Sed quam quam, facilisis vel cursus at.

## Foo
Lorem ipsum dolor sit amet, consectetur adipiscing elit. Proin varius suscipit erat, non porta nunc molestie a.
Nam blandit justo eget volutpat posuere. Etiam in ex fringilla, semper massa posuere, feugiat eros.

## Bar
Vestibulum vel tellus id felis lacinia tristique sollicitudin et lorem. Donec fermentum,
velit nec fringilla consectetur, dolor nibh bibendum velit, sed porta augue eros ut enim.
````

You can also do the same in your other 2 pages, these will serve as the index pages for the `api` and `guide` pages of our documentation app. You can also add other folders and other Markdown files if you like, Dgeni will parse the files the same way as the others and generate HTML partials. The `api` folder (and section) is where your code-generated documentation will be located, Dgeni uses that folder by default.

Now that we've added static content, we need to tell Dgeni where that content is located, and setup the computePathsProcessor so we can output the content to HTML partials. Let's do this now!

First, let's add the Markdown files to the sourceFiles list:

````javascript
// Remember the sourceFiles section we added earlier? Let's now add our .md file to the list !
readFilesProcessor.sourceFiles = [
    { include: 'src/app/**/**/*.js', basePath: 'src/app' }, // All of our application files

    // Our static Markdown documents
    // We are specifying the path and telling Dgeni to use the ngdocFileReader
    // to parse the Markdown files to HTMLs
    { include: 'docs/content/**/*.md', basePath: 'docs/content', fileReader: 'ngdocFileReader' }
];
````

Now, let's setup the computePathsProcessor to output the files to HTML partials:


````javascript
// create new compute for 'content' type doc
computeIdsProcessor.idTemplates.push({
    docTypes: ['content'],
    getId: function(doc) { return doc.fileInfo.baseName; },
    getAliases: function(doc) { return [doc.id]; }
});

// Document this part a little bit more?

// Build custom paths and set the outputPaths for "content" pages
computePathsProcessor.pathTemplates.push({
    docTypes: ['content'],
    getPath: function(doc) {
        var docPath = path.dirname(doc.fileInfo.relativePath);
        if (doc.fileInfo.baseName !== 'index') {
            docPath = path.join(docPath, doc.fileInfo.baseName);
        }
        return docPath;
    },
    outputPathTemplate: 'partials/${path}.html'
});
````


### 5. Basic export to HTML partials

Now that most of the configuration is done, we'll create a Gulp task to generate the documentation. Open up your `gulpfile.js` and add the following

````javascript
var Dgeni = require('dgeni');

gulp.task('dgeni', function() {

    // Notice how we are specifying which config to use
    // This will import the index.js from the /docs/config folder and will use that
    // configuration file while generating the documentation
    var dgeni = new Dgeni([require('./docs/config')]);

    // Using the dgeni.generate() method
    return dgeni.generate();
});
````

Now we can open up the command line, navigate to your project folder and run the Gulp task we wrote above.

````javascript
gulp dgeni
````

This should be the output:

````javascript
[11:06:17] Starting 'dgeni'...
info:    running processor: readFilesProcessor
info:    running processor: extractJSDocCommentsProcessor
info:    running processor: parseTagsProcessor
info:    running processor: filterNgDocsProcessor
info:    running processor: extractTagsProcessor
info:    running processor: codeNameProcessor
info:    running processor: computeIdsProcessor
info:    running processor: memberDocsProcessor
info:    running processor: moduleDocsProcessor
info:    running processor: generateComponentGroupsProcessor
info:    running processor: providerDocsProcessor
info:    running processor: collectKnownIssuesProcessor
info:    running processor: computePathsProcessor
info:    running processor: renderDocsProcessor
info:    running processor: unescapeCommentsProcessor
info:    running processor: inlineTagProcessor
info:    running processor: writeFilesProcessor
info:    running processor: checkAnchorLinksProcessor
[11:06:18] Finished 'dgeni' after 732 ms
````

If you open up the `docs/build` folder you should see the exported HTML partials... but isn't this a little bit boring? They are just plain HTML partials and are not very exciting. We have all these files, but nothing to showcase them! As mentioned previously, Dgeni doesn't come with an application to show the output files, the developer is free to use what he/she wants.

And we are going to do just that! We'll be wrapping up all of our documentation in a simple Angular application.

**One thing to note is that Dgeni doesn't remove the partials while generating the docs. I advise you add another task that will clean up the `/build` folder before it generates the documentation.**

### 6. Creating an Angular app to show the documentation

Now let's create the Angular app that will house and show our HTML partials. Our app will be pretty simple and will only contain:

- Root app module (app.module.js)
- Root app config (app.config.js)
- Docs Controller (docs.js)
- Index view (index.html)
- Bootstrap (for quick styling purposes)

Create `app.module.js` and `app.config.js` under `/docs/app` and add the following.

````javascript
// app.module.js
// Creating the root app module
angular
    .module('docs', []);

// app.config.js
// HTML5Mode to true, adding that config block to our root app module
angular
    .module('docs')
    .config(config);

function config($locationProvider) {

    $locationProvider.html5Mode(true).hashPrefix('!');

}
config.$inject = ["$locationProvider"];
````

We'll get back to the Controller later on, so for now create `docs.js` and insert the following placeholder code

````javascript
angular
    .module('docs')
    .controller('DocsController', DocsController);

function DocsController() {

    var ctrl = this;

    // Controller definition

}

DocsController.$inject = [""];
````

Alright, moving on, we now need to create our index template. Create `indexPage.template.html` under `/docs/config/template`:

````html
<!doctype html>
<html ng-app="docs" ng-controller="DocsController as ctrl">
    <head>
        <base href="/">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <meta charset="utf-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <title>Documentation</title>

        <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css">

    </head>
    <body>


        <nav class="navbar navbar-default">
            <div class="container">

                <div class="navbar-header">
                    <a class="navbar-brand" href="#">Documentation</a>
                </div>

                <ul class="nav navbar-nav">
                    <li class="active"><a href="#">API</a></li>
                    <li><a href="#">Guide</a></li>
                </ul>

            </div>
        </nav>


        <div class="container">

            <div class="row">

                <div class="col-sm-8">

                    <!-- Doc content will go here -->

                </div>

                <div class="col-sm-3 offset-sm-1">

                    <h4>Contents</h4>

                    <!-- Pages for current section -->
                    <ol class="list-unstyled">
                        <li><a href="#"></a></li>
                    </ol>

                </div>

            </div>

        </div>

        <!-- vendors -->
        <script src="//ajax.googleapis.com/ajax/libs/angularjs/1.5.8/angular.min.js"></script>

        <!-- angular app -->
        <script src="src/app.module.js"></script>
        <script src="src/app.config.js"></script>
        <script src="src/docs.js"></script>

        <!-- page data -->
        <script src="src/pages-data.js"></script>
        <script src="src/nav-data.js"></script>
    </body>
</html>
````

We could just add that index file directly in `/app`, but we're actually going to generate the page using a processor we will create ourselves. Dgeni will then run the processor and output our index template.

### 7. Creating a processor for our Angular index page

Now, let's configure Dgeni to compile our index page. Create `index-page.js` under `/docs/config/processors`

````javascript
module.exports = function indexPageProcessor() {
    return {
        $runAfter: ['adding-extra-docs'],
        $runBefore: ['extra-docs-added'],
        $process: process
    };

    function process(docs) {

        // Document what this does? pushes to the docs obj the props
        docs.push({
            docType: 'indexPage',
            template: 'indexPage.template.html',
            outputPath: 'index.html',
            path: 'index.html',
            id: 'index'
        });

    }
};
````

Then, we need to tell Dgeni about this new processor! Let's add it in our config file

````javascript
// /docs/config/index.js
.processor(require('./processors/index-page'))
````

If we run the Gulp task we defined earlier, you'll see this processor added to the pipeline

````javascript
info:    running processor: indexPageProcessor
````

And if you look into your `build` folder, you'll see the processed index.html! That might seem like an unnecessary step, but Dgeni can output all the files we need to display our documentation app in the folder of our choosing. That means that we don't need an external process to copy the index file.

### 8. Generating a list of pages for our sidebar

````javascript
// Create ng constant templates
// Create pages processor
// Add the processor to the pipeline
// Go in NG app and add them to index page
// Go in NG Controller and add constants
// Loop in view
````


### 9. Documenting a Module, Controller, Directive and Service

In the interest of time, I will actually be documenting an existing Angular 1.5 app.

I'm using Todd Motto's (the owner of this blog) Angular 1.5 Component app as a living example. If you haven't had the time to check it out, do so... it's the best example of a .component() based application using UI-Router 1.0 beta using routed components.

[https://github.com/toddmotto/angular-1-5-components-app][dfa622d3]

  [dfa622d3]: https://github.com/toddmotto/angular-1-5-components-app "angular component app"

````javascript
// Briefly show how adding JSDoc and NgDoc tags are going to get parsed... show Module, Service and Methods**
````

### 10. Compile and deploy

````javascript
// Necessary step? maybe split page processor and angular stuff
// Should we talk about copy, concat and minify angular stuff here?
````



### Limitations
Talk about how Controllers and Components cannot be documented correctly and how we could add our own processors to take care of it

### Conclusion

How do we conclude???
