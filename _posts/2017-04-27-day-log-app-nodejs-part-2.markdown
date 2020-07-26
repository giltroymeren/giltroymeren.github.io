---
layout: post
title:  "Day Log App in Node.js Part 2: View DayLog home page"
date:   2017-04-27
categories: projects notes node.js express zombie.js series daylogapp
---

In my [previous post]({{ site.baseurl }}{% post_url 2017-04-24-day-log-app-nodejs-part-1 %}) I laid out the initial structure of the application and implemented the first functional test to create the home page.

This is the start of the [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete) sections that will be discussed further in the following posts the next few days.

## Table of Contents
* TOC
{:toc}

## Development

This post focuses on showing the Day Log home page which should contain a table `DayLog`s if there are available or an information for none otherwise.

### Visit `/daylog` from the home page

The `/daylog` page should show a panel the title "Day Log".

#### Make the failing test

This is a continuation of the test from the previous post. This time, the button "See More" in the Jumbotron should go to the `/daylog` path.

In `/test/functional/index.test.js`, add the following:

~~~ javascript
describe('User clicks "See More" link', () => {
    before((done) => {
        browser.clickLink('.jumbotron a', done);
    });

    it('should go to /daylog', () => {
        browser.assert.success();
        expect(browser.assert.text('.card-header', 'Day Log')).to.be.true;
    });
});
~~~

If we run the test, we expect that it will fail with a `404` error message because the path `/daylog` is not yet setup. However, the following should appear instead about the `.card-header` element:

~~~ console
  User visits home page
    ✓ should contain the jumbotron
    User clicks "See More" link
      1) should go to /daylog


  1 passing (2s)
  1 failing

  1) User visits home page User clicks "See More" link should go to /daylog:
     AssertionError: Expected selector ".card-header" to return one or more elements
      at Assert.text (node_modules/zombie/lib/assert.js:366:7)
      at Context.it (test/functional/index.test.js:28:35)
~~~

To fix this, just change the `href` attribute of the of link in `/views/index.ejs` from `#` to `/daylog` and re-run the test:

~~~ console
  User visits home page
    ✓ should contain the jumbotron
    User clicks "See More" link
      1) "before all" hook for "should contain the jumbotron"


  1 passing (1s)
  1 failing

  1) User visits home page User clicks "See More" link "before all" hook for "should contain the jumbotron":
     Error: Server returned status code 404 from http://localhost:3000/daylog
      at node_modules/zombie/lib/document.js:782:39
      at process._tickCallback (internal/process/next_tick.js:103:7)
~~~

Now it shows a `404` error message.

#### Make the test pass

First `/daylog` should be mapped to a view file.

I chose to rename `/routes/index.js` to `/routes/index.route.js` in order to not be confused with the file names but it is up to you if you want to follow or not; just remember the consistency of the naming convention across the application.

Also change the entry in `app.js` from:

~~~ javascript
var index = require('./routes/index');
...
app.use('/', index);
~~~

to

~~~ javascript
var indexRoutes = require('./routes/index.route');
...
app.use('/', indexRoutes);
~~~

Next, create `/routes/daylog.route.js`:

~~~ javascript
'use strict';

var express = require('express');
var router = express.Router();

router.get('/', (request, response) => {
    response.render('daylog/index');
});

module.exports = router;
~~~

The path in `get` will map to the view file `daylog/index.ejs`. However, we already have `/` set to `index` in `app.js`. To register all paths starting with `daylog` to this file, just add the following in `app.js`, similar to `index.route.js`:

~~~ javascript
var daylogRoutes = require('./routes/daylog.route');
...
app.use('/daylog', daylogRoutes);
~~~

Now create the page in `/views/daylog/index.ejs`:

~~~ html
<% include ../partials/header %>

<div class="card">
    <h3 class="card-header">Day Log</h3>
</div><!-- .card -->

<% include ../partials/footer %>
~~~

#### Run the tests

The test case `should go to /daylog` should pass:

~~~ console
  User visits home page
    ✓ should contain the jumbotron
    User clicks "See More" link
      ✓ should go to /daylog


  2 passing (4s)
~~~

### Visit `/daylog` when there are no `DayLog`s available

This is the start of the `/daylog` page specific test creation. As mentioned before, there are two cases when visiting this page depending on the existence of `DayLog`s in the database. The first is when there are none, which makes sense since we have not implemented the creation part yet.

You can use any database for this part and I chose [mongodb](https://www.mongodb.com/) because of its simplicity to setup and use out-of-the-box.

#### Make the failing test

Each time the `/daylog` page is accessed, it should show all `DayLog`s from the database so we need a way to connect. For mongodb, the [`mongoosejs`](http://mongoosejs.com/) library is used. Please install it to the project via:

    npm install mongoose --save

Next, create a new test file `/test/functional/daylog.test.js`:

~~~ javascript
'use strict';

var expect = require('expect.js');
var Browser = require('zombie');
var browser = new Browser();
var mongoose = require('mongoose');

var DayLogModel = require('../../model/daylog.model');
var constants = require('../../config/constants');

const BASE_URL = constants.BASE_URL + '/daylog';

describe('User visits /daylog', () => {
    before(() => {
        mongoose.connect(constants.DATABASE.testing);
    });

    beforeEach((done) => {
        browser.visit(BASE_URL, done);
    });

    it('should contain the Day Log section', () => {
        browser.assert.success();
        browser.assert.text('title', 'DayLogApp');
        expect(browser.assert.text('.card-header', 'Day Log')).to.be.true;
    });

    it('should show an alert when no DayLogs are available', () => {
        browser.assert.success();
        expect(browser.assert.element('.alert-info')).to.exist;
    });

    afterEach((done) => {
        mongoose.connection.dropDatabase();
        done();
    });
});
~~~~

Basically the test case `should contain the Day Log section` is to confirm the test from `index.route.js` for the panel.

`should show an alert when no DayLogs are available`, on the other hand, expects a page with a "Day Log" panel and an alert message with class `.alert-info` (I chose to check the class because the message may change and classes are shorter and easier to change).

![DayLog index page - empty table](/assets/images/posts/2017-04-27-day-log-app-nodejs-part-2/daylog-table-empty.png){:class="img-responsive"}

The `before`, `beforeEach` and `afterEach` are setup and teardown helper methods. `before` will be executed *before* any `beforeEach` and `it` blocks, so it makes sense to put the connection there. `beforeEach`, as the name implies, is run before each `it` block and the command to visit the `BASE_URL`, the link to the page, is set there. Finally, `afterEach` is run after each `it` block and I chose to clear the database each time a test case is performed to make sure they use a blank slate.

##### Create constants file

If you noticed, there is a `constants` variable in the file. I moved all reusable, constant variables there to make sure they are not duplicated in the files.

I just made `/config/constants.js` with the following content:

~~~ javascript
'use strict';

module.exports = Object.freeze({
    DATABASE: {
        production: 'mongodb://localhost/daylogapp',
        testing: 'mongodb://localhost/daylogapp-test'
    },
    ENVIRONMENT: {
        production: 'production',
        testing: 'testing'
    },
    BASE_URL: 'http://localhost:3000'
});
~~~

Thanks to this, I have set the database connection to use `constants.DATABASE.testing` only during this test.

#### Make the test pass

The routes page before just shows the file `/daylog/index.ejs` when `/daylog` is visited. Now we have a requirement to fetch `DayLog`s from the database and show them in that file.

Modify `/routes/daylog.route.js` and add the following:

~~~ javascript
var database = require('../config/database');

var DayLogModel = require('../model/daylog.model');
var DayLogController = require('../controller/daylog.controller');

router.get('/', DayLogController.index);
~~~

The changes are as follows:

*   We add a database connection setup to `/config/database.js`:

    ~~~ javascript
    'use strict';

    var mongoose = require('mongoose');
    var constants = require('./constants');

    var databaseURL = constants.DATABASE.production;

    if(process.env.NODE_ENV === constants.ENVIRONMENT.testing) {
        databaseURL = constants.DATABASE.testing;
    }

    mongoose.connect(databaseURL);
    mongoose.connection.on('connection', () => {
        console.log('Mongoose default connection open to: ' + databaseURL);
    });

    module.exports = mongoose;
    ~~~

    Here, `process.env.NODE_ENV` refers to the `NODE_ENV` variable that should be declared whenever the application is run:

        NODE_ENV=testing node app.js

    I put `NODE_ENV` in the front in case the `node` command and other options after it requires a path.

    I chose to add this feature in order to set different configurations between production and testing development. From the constants file, I declared two different databases for each respectively: `mongodb://localhost/daylogapp` and `mongodb://localhost/daylogapp-test`.

    *   Reminder to check if the databases exist. Prepare two terminal windows: one to run `mongodb` and one to access it (this is how my installation works; MySQL however has a GUI to run it and only one terminal to access via shell):

        Run `mongodb` with the command `mongod` in one terminal and access its shell with `mongo`.

        Next, check if they exist with `show dbs` and if they do not, create them with `use <database-name>`.

*   The database needs a structure, so a model for `DayLog` is needed.

    In `/model/daylog.model.js`:

    ~~~ javascript
    'use strict';

    var mongoose = require('mongoose');
    var DayLogSchema = new mongoose.Schema({
        title: {
            type: String,
            required: true
        },
        slug: {
            type: String,
            required: true,
            unique: true
        },
        location: {
            type: String,
            required: true
        },
        logAt: {
            type: Date,
            required: true
        },
        category: {
            type: String,
            enum: ['Adequate', 'Minor', 'Major'],
            required: true,
        },
        updatedAt: {
            type: Date,
            default: Date.now,
        }
    });

    var DayLogModel = mongoose.model('DayLog', DayLogSchema);

    module.exports = DayLogModel;
    ~~~

    This is the same structure as the original [DayLogApp]({{ site.baseurl }}{% post_url 2017-03-20-day-log-app %}) with the `slug` as the unique key. I set this because I do not want to expose the actual database ID in the URL.

*   Instead of an anonymous method to show the file:

    ~~~ javascript
    router.get('/', (request, response) => {
        response.render('daylog/index');
    });
    ~~~

    We move all route methods in a controller to separate the content and responsibility from the routes.

    This is `/controller/daylog.controller.js`:

    ~~~ javascript
    'use strict';

    var DayLogModel = require('../model/daylog.model');

    var DayLogController = {
        index: (request, response) => {
            DayLogModel.find({},
                (err, daylogs) => {
                    if(err) return console.error(err);

                    return response.format({
                        html: () => {
                            response.render('daylog/index', { daylogs: daylogs });
                        },
                        json: () => { response.json(daylogs); }
                    })
                }
            );
        },
    };

    module.exports = DayLogController;
    ~~~

    Since `DayLogModel` in an instance of `mongodb` via `mongoose`, we can use its [native collection query methods](https://docs.mongodb.com/manual/reference/method/js-collection/) directly.

    Here, `find({})` takes in a criteria and since the empty object is provided, it will fetch all.

    Next, we just set the view file to be `daylog/index` (automatically searches the `views` folder) with variable `daylogs` that will be accessible to the page via EJS.

Next add the following to `/views/daylog/index.ejs`:

~~~ html
<% if(daylogs.length) { %>
<pre><%= daylogs %></pre>
<% } else { %>
<div class="card-block">
    <div class="alert alert-info" role="alert">
        There are no Day Logs available. <a href="#" class="alert-link">Create</a> one now!
    </div>
</div>
<% } %>

<div class="card-footer text-center">
    <a href="/create" class=" btn btn-primary">Create Day Log</a>
</div>
~~~

This time, depending on the length of `daylog` (from the controller), if more than one, will just show everything as is with `pre` and otherwise the message that there are none available.

The create button is already added to complete the look and it will be dealt with later :-)

#### Run the tests

~~~ console
  User visits /daylog
    ✓ should contain the Day Log section
    ✓ should show an alert when no DayLogs are available


  2 passing (2s)
~~~

### Visit `/daylog` when there are `DayLog`s available

In the previous test we left the structure of the available `DayLog`s in the view to just show as is with the `pre` element. Now they should be in a table similar to the original.

![DayLog index page - not empty table](/assets/images/posts/2017-04-27-day-log-app-nodejs-part-2/daylog-table.png){:class="img-responsive"}

#### Make the failing test

In the test file, declare a new `DayLog` to be created during the test:

~~~ javascript
...
const BASE_URL = constants.BASE_URL + '/daylog';
const DAYLOG = new DayLogModel({
    "title" : "Lorem ipsum dolor sit amet",
    "slug" : "lorem-ipsum",
    "location" : "Nec congue lorem nulla id ligula",
    "logAt" : Date.now(),
    "category" : "Major"
});
...
~~~

and the test to check the table:

~~~ javascript
describe('Insert one Day Log in database', () => {
    before(() => {
        DayLogModel.create(DAYLOG);
    });

    beforeEach((done) => {
        browser.visit(BASE_URL, done);
    });

    it('should show the Day Log in a table when it is added', () => {
        browser.assert.success();
        expect(browser.assert.element('.card')).to.exist;
        expect(browser.assert.text('.card-header', 'Day Log')).to.be.true;
        expect(browser.assert.element('table')).to.exist;
    });
});
~~~

We need to visit `/daylog` again in this case because, as said before, the page fetches from the database each view and to reflect the just-created `DayLog` it should be refreshed. This is not needed if it is added automatically without refresh using AJAX but that is reserved for future improvements.

#### Make the test pass

Implement the table in `/views/daylog/index.ejs` by replacing:

~~~ html
<pre><%= daylogs %></pre>
~~~

with

~~~ html
...
<% if(daylogs.length) { %>
<table class="table">
    <thead>
        <tr>
            <th>Title</th>
            <th>Created</th>
            <th>Updated</th>
            <th>Tasks</th>
            <th></th>
            <th></th>
        </tr>
    </thead>

    <tbody>
    <% daylogs.forEach((daylog) => { %>
        <tr>
            <td>
                <a href="/<%= daylog.slug %>">
                    <strong><%= daylog.title %></strong></td>
                </a>
            <td>
                <code><%= daylog._id.getTimestamp().toISOString().substring(0, 10) %></a>
                </code>
            </td>
            <td>
                <code><%= daylog.updatedAt.toISOString().substring(0, 10) %></code>
            </td>
            <td><code>0</code></td>
            <td>
                <a href="/<%= daylog.slug %>/edit" class="btn btn-primary btn-sm">Edit</a>
            </td>
            <td>
                <form action="/<%= daylog.slug %>/delete" method="POST">
                    <input type='hidden' value='DELETE' name='_method'>
                    <button type="submit" class="btn btn-sm btn-danger">Delete</button>
                </form>
            </td>
        </tr>
    <% }); %>
    </tbody>
</table>
<% } else { %>
...
~~~

`mongodb` dates are of the format [UTC](https://www.timeanddate.com/time/aboututc.html) and these need to be converted to the format `DD-MM-YYYY` to match the `date` HTML element.

The tasks, edit and delete buttons for each `DayLog` are also available which will all be implemented in the later articles.

#### Run the tests

~~~ console
  User visits /daylog
    ✓ should contain the Day Log section
    ✓ should show an alert when no DayLogs are available
    Insert one Day Log in database
      ✓ should show the Day Log in a table when it is added


  3 passing (6s)
~~~

## References
* Kim, Kay. "Date()." *Date() — MongoDB Manual 3.4*. N.p., 19 May 2016. Web. 27 Apr. 2017. <[`https://docs.mongodb.com/manual/reference/method/Date/`](https://docs.mongodb.com/manual/reference/method/Date/)>.
* Rhodes, Tamlyn. "How to reset test database before each test run? · Issue #44 · dwyl/the-book." *GitHub*. N.p., 25 June 2015. Web. 25 Apr. 2017. <[`https://github.com/dwyl/the-book/issues/44#issuecomment-115186213`](https://github.com/dwyl/the-book/issues/44#issuecomment-115186213)>.
* Yaapa, Hage. "Running Express.js in Production Mode." *Hack Sparrow*. N.p., 21 Feb. 2012. Web. 24 Apr. 2017. <[`https://www.hacksparrow.com/running-express-js-in-production-mode.html`](https://www.hacksparrow.com/running-express-js-in-production-mode.html)>.

