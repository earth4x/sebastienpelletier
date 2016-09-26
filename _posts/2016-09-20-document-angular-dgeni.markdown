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
- <a href="#installing-and-configuration">5. Basic export to HTML partials</a>
- <a href="#installing-and-configuration">6. Creating an Angular app to show the documentation</a>
- <a href="#installing-and-configuration">7. Creating a processor for our Angular index page</a>
- <a href="#installing-and-configuration">8. Generating a list of pages for our sidebar</a>
- <a href="#installing-and-configuration">9. Documenting a Module, Controller, Directive and Service</a>
- <a href="#installing-and-configuration">10. Compile and deploy</a>
- <a href="#documenting-your-code">Documenting your code</a>
- <a href="#generating-documentation">Generating documentation</a>
- <a href="#how-to-improve">How to improve</a>
- <a href="#limitations">Limitations</a>

Set this up as a tutorial so that people can follow along... Setup Angular's documentation generator for your Angular app in 10 steps

People also like numbers, steps to follow - so by the end they can do it all themselves for their app. An example of "how" to actually document the app would also be *amazing*!
Todd Motto

Step 9) Documenting the app
Now we need to go to some code examples and do this, this and this.
Todd Motto

Step 10) Final compile and deploy


### What's Dgeni

Dgeni (pronounced Jenny) is an extremely powerful NodeJS documentation generation utility. It was built by people from the Angular team, so it does make everything easy to document Angular projects although it can be used for other projects as well. AngularJS, Ionic Framework, Protractor and Angular Material are currently using Dgeni in production to generate their documentation.

The main feature of Dgeni is to convert your source files to documentation files. You can convert them to a full HTML page, partials, or something else. It has a modular core and offers multiple plugins (packages). You configure your base package using Processors and Services to customize how you want to generate your documentation.

All of your documentation needs to be written in a form of JSDoc, which is a standard for Javascript documentation (although ESDoc is picking up!), and is written inline in the source code. The various Dgeni Processors will then scan and convert your source files to documentation files. This also means that it supports many of the standard JSDoc directives, so if you are already writting JSDoc today, your work won't be lost :)

Dgeni does not provide a web application to display the output files, but it allows you to do what you want with it. It's up to the developer to decide and n this example we'll actually wrap our documentation in a simple Angular app.

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
**Talk about using gulp... and make a comment about cleaning up the docs folder because Dgeni doesn't remove the partials (no cleaning)**

First thing we are going to do is create a `docs` folder where we will have our configuration files, our static content as well as the actual documentation. In this case, I opted to simply put it in the same directory under `build`, but you could package it in a `dist` folder if you want.

**SETUP GULP TASK**

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
Now, let's open up `index.js`, and let's start configuring this beast !

** Note if you are using an older LoDash version, keyBy is indexBy

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
.config(function(dgeni, log, readFilesProcessor, writeFilesProcessor) {

    // Set the log level to 'info', switch to 'debug' when troubleshooting
    log.level = 'info';

    // Specify the base path used when resolving relative paths to source and output files
    readFilesProcessor.basePath = path.resolve(packagePath, '../..');

    // Specify our source files that we want to extract
    readFilesProcessor.sourceFiles = [
        { include: 'src/app/**/**/*.js', basePath: 'src/app' },
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

    // Here we are defining our docType Module
    //
    // Each angular module will be extracted to it's own partial
    // and will act as a container for the various Components, Controllers, Services in that Module
    computePathsProcessor.pathTemplates.push({
        docTypes: ['module'],
        pathTemplate: '${area}/${name}',
        outputPathTemplate: 'partials/${area}/${name}.html'
    });

    // Doing the same thing but for regular types like Services, Controllers, etc...
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

````javascript

// create new processor for 'content' type doc
// add 'overview' parsing to config file
// add .ngdoc file for /api overview
// add file for guide overview
// add file for other guide content

````

### 5. Basic export to HTML partials

````javascript

// Setup a gulp task
// Run the generate command
// Show 'boring' result

````

### Documenting your code

In the interest of time, I will actually be documenting an existing Angular 1.5 app.

I'm using Todd Motto's (the owner of this blog) Angular 1.5 Component app as a living example. If you haven't had the time to check it out, do so... it's the best example of a .component() based application using UI-Router 1.0 beta using routed components.

[https://github.com/toddmotto/angular-1-5-components-app][dfa622d3]

  [dfa622d3]: https://github.com/toddmotto/angular-1-5-components-app "angular component app"

**Briefly show how adding JSDoc and NgDoc tags are going to get parsed... show Module, Service and Methods**

### Generating Documentation

Using the Gulp task, looking at the partials folder (screenshot)

Looking at the partials folder is very boring. They are just plain HTML partials and are not very exciting. As mentioned at the beginning of this article, Dgeni doesn't provide a `wrapper` application to display the partials. It's up to each individual to present it how he/she wants.

And we are going to do just that! We'll be wrapping up all of our documentation in a simple Angular application.

### How to improve

- Wrapping everything in an Angular App
- Adding a new processor to generate an object containing all of our pages
- Adding a new processor to generate an index page for our app
- Modifying our config file to add these new processors

````javascript
// Adding our index page processor
.processor(require('./processors/index-page'))

// Adding our page-data processor
.processor(require('./processors/pages-data'))
````

Then, we can add our static .ngdoc files to our source files

````javascript
readFilesProcessor.sourceFiles = [
    { include: 'client/app/**/**/*.js', basePath: 'client/app' },
    { include: 'docs/content/**/*.ngdoc', basePath: 'docs/content' } // newly added
];
````


### Limitations
Talk about how Controllers and Components cannot be documented correctly and how we could add our own processors to take care of it

### Conclusion

How do we conclude???
