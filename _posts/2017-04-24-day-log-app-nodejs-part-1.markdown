---
layout: post
title:  "Day Log App in Node.js Part 1: Home page"
date:   2017-04-24
categories: projects notes node.js express zombie.js series daylogapp
---

This is the first part of a series of my attempt to implement my previous Laravel project ["Day Log App"](http://daylogapp.herokuapp.com/) in Node.js. I made a post about customizing an [Express site with EmbeddedJS]({{site_url}}{% post_url 2017-04-22-how-to-change-express-jade-to-ejs %}) and this is actually the *prologue* to this project. I will post the other parts within the following weeks as I successfully implement my personal feature goals for each article.

In the spirit of my [original Laravel article]({{ site.baseurl }}{% post_url 2017-03-20-day-log-app %}) each section follows the TDD/BDD development ([functional tests](https://en.wikipedia.org/wiki/Functional_testing) in this case).

## Table of Contents
* TOC
{:toc}

## Steps

### Install dependencies

We will use the browser testing library [zombie.js](http://stackoverflow.com/questions/9350232/mocha-and-zombiejs/9444550#9444550) to test our application's web pages' behavior.

    > npm install zombie --save-dev
    > npm install expect.js --save-dev

### First functional test case

Our test will visit `http://localhost:3000/` and should expect the following page:

![DayLogApp Node.js index page](/assets/images/posts/2017-04-24-day-log-app-nodejs-part-1/index-page.png){:class="img-responsive"}


Create the `test` folder. This is the default location where [Mocha](https://mochajs.org/), the JavaScript testing framework Zombie is based on, will search for test files. Since we are interested in functional tests, create the folder `functional` inside it and `index.test.js` as our first test file.

~~~ javascript
'use strict';

process.env.NODE_ENV = 'test';

var expect = require('expect.js');
var Browser = require('zombie');
var browser = new Browser();

const BASE_URL = 'http://localhost:3000/';

describe('User visits home page', () => {
    before((done) => {
        browser.visit(BASE_URL, done);
    });

    it('should contain the jumbotron', () => {
        browser.assert.success();
        browser.assert.text('title', 'DayLogApp');
        expect(browser.assert.element('.jumbotron')).to.exist;
    });

    after((done) => {
        done();
    });
});
~~~

`browser` refers to the environment where Zombie will perform our tests given by the `visit()` method. This task is placed inside the `before` method (or setup) as [recommended by the creator](https://github.com/assaf/zombie/issues/903#issuecomment-95749229):

>  Use `before` blocks for navigation (`browser.visit`, `clickLink`, etc), `it` blocks for assertions.

`browser.assert.success();` is added to make sure the page is loaded with status `200`.

`.jumbotron` refers to the splash page element I decided to include. It is simply lifted from the [Bootsrap 4 documentations](https://v4-alpha.getbootstrap.com/components/jumbotron/).

Running `mocha /test/functional/index.test.js` should fail page we have not modified the `.ejs` view files.

### Make the test pass

#### Update routes

The routes used in the application are declared in `app.js` as:

    var indexRoutes = require('./routes/index.route');
    ...
    app.use('/', indexRoutes);

Modify `/routes/index.route.js` with the following:

~~~ javascript
'use strict';

var express = require('express');
var router = express.Router();

/* GET home page. */
router.get('/', function(req, res, next) {
  res.render('index', { title: 'DayLogApp' });
});

module.exports = router;
~~~


#### Update views

First delete the file `/views/layout.ejs` as it has no use in this project.

Prepare the header and footer for the entire project:

*   `/views/partials/header.ejs`

    This project will just use Bootstrap 4 loaded online.

    ~~~ html
    <!DOCTYPE html>
    <html lang="en">
        <head>
            <meta name="viewport" content="width=device-width, initial-scale=1">

            <title>DayLogApp</title>

            <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0-alpha.6/css/bootstrap.min.css">
        </head>

        <body>
            <nav class="navbar navbar-toggleable-md navbar-light bg-faded"
                style="margin-bottom: 2em">
                <div class="container col-sm-6">
                    <button class="navbar-toggler navbar-toggler-right" type="button"
                        data-toggle="collapse" data-target="#navbarSupportedContent"
                        aria-controls="navbarSupportedContent" aria-expanded="false"
                        aria-label="Toggle navigation">
                        <span class="navbar-toggler-icon"></span>
                    </button>
                    <a class="navbar-brand" href="#">DayLogApp</a>
                </div>
            </nav>

            <div class="row">
                <div class="container col-sm-6">
    ~~~

*   `/views/partials/footer.ejs`

    ~~~ html
                </div><!-- .container -->
            </div><!-- .row -->

            <script src="https://code.jquery.com/jquery-3.1.1.slim.min.js"></script>
            <script src="https://cdnjs.cloudflare.com/ajax/libs/tether/1.4.0/js/tether.min.js"></script>
            <script src="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0-alpha.6/js/bootstrap.min.js"></script>
        </body>
    </html>
    ~~~~

Next is the splash page in `/views/index.ejs`

~~~ html
<% include partials/header %>

<div class="jumbotron">
    <h1 class="display-3">Day Log App</h1>
    <p class="lead">Manage your daily logs and their tasks.</p>
    <hr class="my-4">
    <p class="lead">
        <a class="btn btn-info btn-lg" href="#" role="button">See more</a>
    </p>
</div>

<% include partials/footer %>
~~~

Optional (but recommended) is the error page (`/views/error.ejs`) to display helpful information in case of trouble, especially during testing.

~~~ html
<% include partials/header %>

<h2>An error occurred.</h2>
<p><%= message %></p>
<p><%= error.status %></p>
<code><%= error.stack %></code>

<% include partials/footer %>
~~~

### Run test

To confirm that our index page has been setup correctly, run the test:

    > mocha /test/functional/index.test.js

and it should pass!

      User visits home page
        ✓ should contain the jumbotron

      1 passing (1s)

#### Timeout

In case the test fails with the following timeout message:

    > Error: Timeout of <time> exceeded. For async tests and hooks, ensure "done()" is called; if returning a Promise, ensure it resolves.

add the a time parameter `-t` to the execution and try which value accommodates the timeout issue:

    > mocha /test/functional/index.test.js -t 5000

## References

* Assaf Arkin. "Can we do the assertion in the callback of `browser.visit` as the document shows? · Issue #903 · assaf/zombie." *GitHub*. N.p., 24 Apr. 2015. Web. 24 Apr. 2017. <[`https://github.com/assaf/zombie/issues/903#issuecomment-95749229`](https://github.com/assaf/zombie/issues/903#issuecomment-95749229)>.
* Industrial. "Mocha and ZombieJS." *Node.js - Mocha and ZombieJS - Stack Overflow*. N.p., 12 Feb. 25. Web. 24 Apr. 2017. <[`http://stackoverflow.com/a/9444550`](http://stackoverflow.com/a/9444550)>.