---
layout: post
title:  "Day Log App in Node.js Part 3: Create a DayLog"
date:   2017-04-29
categories: projects notes node.js express zombie.js series daylogapp
---

This post will discuss how to implement the page to create a `DayLog`.

## Table of Contents
* TOC
{:toc}

## Development

### Visit `/daylog/create` from "Day Log" home page

#### Make the failing test

The test in the previous view test cases start in the `/daylog` page, the "Create Day Log" page is available at the bottom of the panel that will go to the create page.

Add the following after the `Insert one Day Log in database` `describe` block:

~~~ javascript
describe('Clicks "Create Day Log" button', () => {
    beforeEach((done) => {
        browser.clickLink('.card-footer .btn', done);
    });

    it('should go to /daylog/create', () => {
        browser.assert.success();
        browser.assert.url(BASE_URL + '/create');
        expect(browser.assert.element('form')).to.exist;
    });
});
~~~

The URL must correspond to `/daylog/create` and a `form` to be present.

It should fail because the route to `/create` is not yet implemented:

~~~ console
  User visits /daylog
    ✓ should show an alert when no DayLogs are available
    Insert one Day Log in database
      ✓ should show the Day Log in a table when it is added
    Clicks "Create Day Log" button
      1) should go to /daylog/create


  2 passing (14s)
  1 failing

  1) User visits /daylog Clicks "Create Day Log" button should go to /daylog/create:

      AssertionError: 'http://localhost:3000/daylog' deepEqual 'http://localhost:3000/daylog/create'
      + expected - actual

      -http://localhost:3000/daylog
      +http://localhost:3000/daylog/create

      at assertMatch (node_modules/zombie/lib/assert.js:24:212)
      at Assert.url (node_modules/zombie/lib/assert.js:83:9)
      at Context.it (test/functional/daylog.test.js:67:28)
~~~

#### Make the test pass

First the link to the create page in the home page is incorrect. Update the following link in `/views/daylog/index.ejs` from

~~~ html
<a href="/create" class=" btn btn-primary">Create Day Log</a>
~~~

to

~~~ html
<a href="/daylog/create" class=" btn btn-primary">Create Day Log</a>
~~~

Next is to register this route in `/routes/daylog.router.js`:

~~~ javascript
router.get('/create', DayLogController.create);
~~~

Together with its controller method:

~~~ javascript
create: (req, res) => {
    res.render('daylogs/form', {
        operation: 'Create',
        method: 'post',
        action: 'store'
    });
}
~~~~

The attributes `method` and `action` are the `form`'s respective attributes of the same names in the view and `operation` is for the panel header i.e. "Create Day Log".

The form is similar to the original however I changed the "Category" input from a dropdown to radio buttons (I was reading a UX article that emphasized in limiting the pressing a user has to do in a form).

![DayLog Create page - default appearance](/assets/images/posts/2017-04-29-day-log-app-nodejs-part-3/daylog-form-default.png){:class="img-responsive"}

Create a new file `/views/daylog/form.ejs`:

~~~ html
<% include ../partials/header %>

<div class="card">
    <h3 class="card-header"><%= operation %> Day Log</h3>

    <div class="card-block">

        <form action="<%= action %>" method="<%= method %>" enctype="application/x-www-form-urlencoded">

            <div class="container">
                <div class="form-group row">
                    <label for="title" class="col-sm-2 col-form-label text-right">Title</label>
                    <div class="col-sm-10">
                        <textarea class="form-control" id="title" name="title"
                            placeholder="Describe your log"></textarea>
                    </div>
                </div>

                <div class="form-group row">
                    <label for="slug" class="col-sm-2 col-form-label text-right">Slug</label>
                    <div class="col-sm-10">
                        <input type="text" class="form-control" id="slug" name="slug"
                            placeholder="example-daylog-slug">
                    </div>
                </div>

                <div class="form-group row">
                    <label for="location" class="col-sm-2 col-form-label text-right">Location</label>
                    <div class="col-sm-10">
                        <input type="text" class="form-control" id="location" name="location"
                            placeholder="Location of log">
                    </div>
                </div>

                <div class="form-group row">
                    <label for="logAt" class="col-sm-2 col-form-label text-right">Date of log</label>
                    <div class="col-sm-10">
                        <input type="date" class="form-control" id="logAt" name="logAt"
                            placeholder="Date of log">
                    </div>
                </div>

                <div class="form-group row">
                    <label class="col-sm-2 col-form-label text-right">Category</label>
                    <div class="col-sm-10">
                        <div class="form-check">
                            <label class="form-check-label">
                                <input class="form-check-input" type="radio"
                                    name="category" id="category" value="Adequate">
                                Adequate
                            </label>
                        </div>
                        <div class="form-check">
                            <label class="form-check-label">
                                <input class="form-check-input" type="radio"
                                    name="category" id="category" value="Minor">
                                Minor
                            </label>
                        </div>
                        <div class="form-check">
                            <label class="form-check-label">
                                <input class="form-check-input" type="radio"
                                    name="category" id="category" value="Major">
                                Major
                            </label>
                        </div>
                    </div>
                </div>

                <div class="form-group row">
                    <div class="offset-sm-2 col-sm-10">
                        <button type="submit" class="btn btn-primary">Submit</button>
                        <button type="reset" class="btn btn-danger">Reset</button>
                    </div>
                </div>
            </div><!-- .container -->

        </form>

    </div><!-- .card-block -->

    <div class="card-footer text-center">
        <a href="/daylog" class=" btn btn-primary btn-sm">Back</a>
    </div>

</div><!-- .card -->


<% include ../partials/footer %>
~~~

#### Run the tests

~~~ console
  User visits /daylog
    ✓ should show an alert when no DayLogs are available
    Insert one Day Log in database
      ✓ should show the Day Log in a table when it is added
    Clicks "Create Day Log" button
      ✓ should go to /daylog/create


  3 passing (5s)
~~~

### Submit invalid form with validation errors

This time we need to implement validation for the form's fields. The [Express](https://expressjs.com/) framework has a [built-in validation](https://github.com/ctavan/express-validator) and this is used to check the incoming data from the form.

First we should add this validator as a dependency:

    npm install express-validator --save

Register it to be used by the routers in `/routes/daylog.route.js`:

~~~ javascript
var expressValidator = require('express-validator');
...
router.use(expressValidator());
~~~

#### Make the failing test

We will define a second `describe` block just for the creation tests. To be honest I was going to keep everything in one block but I had difficulty in the later tests and this was the one that worked.

~~~ javascript
describe('User visits /daylog/create', () => {
    before((done) => {
        browser.visit(BASE_URL + '/create', done);
    });

    describe('Clicks submit button with empty form', () => {
        beforeEach((done) => {
            browser.pressButton('button[type="submit"]', done);
        });

        it('should show 5 "required" error messages near the 5 fields', () => {
            browser.assert.elements('form .alert-danger', 5);
            browser.assert.text('form .alert-danger', new RegExp('required$'));
        });
    });

    afterEach((done) => {
        mongoose.connection.dropDatabase();
        done();
    });
});
~~~

The test expects that the same page with error alert messages near the fields after submitting the form.

![DayLog Create page - empty form validation errors](/assets/images/posts/2017-04-29-day-log-app-nodejs-part-3/daylog-form-error-required.png){:class="img-responsive"}

The assertion below conveniently checks for the message pattern if they contain the word "required" at the end, which is what will be set in the validation later.

~~~ javascript
browser.assert.text('form .alert-danger', new RegExp('required$'));
~~~

It should fail mainly because the path `/daylog/store`, the submission path, is not yet defined.

~~~ console
  User visits /daylog
    ✓ should show an alert when no DayLogs are available
    Insert one Day Log in database
      ✓ should show the Day Log in a table when it is added
    Clicks "Create Day Log" button
      ✓ should go to /daylog/create

  User visits /daylog/create
    Clicks submit button with empty form
      1) "before each" hook for "should show 5 "required" error messages near the 5 fields"


  3 passing (22s)
  1 failing

  1) User visits /daylog/create Clicks submit button with empty form Clicks submit button with empty form "before each" hook for "should show 5 "required" error messages near the 5 fields":
     Error: Server returned status code 404 from http://localhost:3000/daylog/store
      at node_modules/zombie/lib/document.js:782:39
      at process._tickCallback (internal/process/next_tick.js:103:7)
~~~

#### Make the test pass

Add new the route path:

~~~ javascript
router.post('/store', DayLogController.store);
~~~

Its controller method:

~~~ javascript
store: (req, res) => {
    validateDayLog(req);
    var newDayLog = getDayLogFromRequest(req);

    req.asyncValidationErrors()
        .then(() => {

        })
        .catch((errors) => {
            return res.format({
                html: () => {
                    res.render('daylogs/form', Object.assign({
                            daylog: newDayLog,
                            errors: errors
                        }, getFormPageConfig(constants.FORM_OPERATIONS.create))
                    );
                }
            });
        });
}
~~~

`req.asyncValidationErrors()` checks for the result of the assertions made. If there are no errors, it proceeds to the `then` block or `catch` otherwise. In the `catch` block, it just sends back the error messages as `errors` back to the same view file.

*   The incoming `request` is validated with `validateDayLog`:

    ~~~ javascript
    function validateDayLog(request, operation = null) {
        request.assert('title', 'Title is required').notEmpty();
        request.assert('slug', 'Slug is required').notEmpty();
        request.assert('location', 'Location is required').notEmpty();
        request.assert('logAt', 'Log date is required').notEmpty();
        request.assert('category', 'Category is required').notEmpty();
    }
    ~~~

*   I also added a helper method to get a `DayLogModel` instance from the `request`:

    ~~~ javascript
    function getDayLogFromRequest(request) {
        return new DayLogModel({
            title : request.body.title,
            slug : request.body.slug,
            location : request.body.location,
            logAt : (request.body.logAt) ? new Date(request.body.logAt) : '',
            category : request.body.category
        });
    }
    ~~~

*   The method `getFormPageConfig` just assembles the custom `form` attributes:

    ~~~ javascript
    function getFormPageConfig(operation, daylog = null) {
        var config = {
            operation: operation,
            constants: constants,
            method: constants.HTTP_METHODS.post
        }

        switch (operation) {
            case constants.FORM_OPERATIONS.create:
                config.action = "store";
                break;
        }

        return config;
    }
    ~~~

    This also affects the `create` method so change it from:

    ~~~ javascript
    create: (req, res) => {
        res.render('daylog/form', {
            operation: 'create',
            method: 'post',
            action: 'store'
        });
    }
    ~~~

    to

    ~~~ javascript
    create: (req, res) => {
        res.render('daylog/form',
            getFormPageConfig(constants.FORM_OPERATIONS.create));
    }
    ~~~

*   In relation, add these new constants in `/config/constants/js`:

    ~~~ javascript
    FORM_OPERATIONS: {
        create: 'Create'
    },
    HTTP_METHODS: {
        post: 'post'
    },
    ~~~

    These will be updated in the coming CRUD operations just for the user interface panel title.

    Do not forget to import it in the file:

    ~~~ javascript
    var constants = require('../config/constants');
    ~~~

Next the form page should be updated with the alert messages; here is a snippet:

~~~ html
<% if(typeof errors !== 'undefined' &&
        errors.find(error => error.param === 'title')) { %>
    <%- include('../partials/alert', { errors: errors, type: 'alert-danger',
        field: 'title' }) %>
<% } %>
~~~

These are placed inside each `<div class="col-sm-10">` block directly after the HTML `input` fields.

This just checks if there are errors, the alert message will be called in `/views/partials/alert.ejs`:

~~~ html
<div class="alert <%= type %>" role="alert">
    <button type="button" class="close" data-dismiss="alert" aria-label="Close">
        <span aria-hidden="true">&times;</span>
    </button>
    <%= errors.find(error => error.param === field).msg %>
</div>
~~~

#### Run the tests

~~~ console
  User visits /daylog
    ✓ should show an alert when no DayLogs are available
    Insert one Day Log in database
      ✓ should show the Day Log in a table when it is added
    Clicks "Create Day Log" button
      ✓ should go to /daylog/create

  User visits /daylog/create
    Clicks submit button with empty form
      ✓ should show 5 "required" error messages near the 5 fields


  4 passing (9s)
~~~

### Submit valid form

A valid form has its all fields answered with the proper formats. This section just shows the OK scenario for a valid `DayLog` creation. The other specific validations will be discussed in the next section.

#### Make the failing test

The test will fill all the fields with valid information and after submission, the user should be directed to this `DayLog`'s view page and it should contain the same data as the submitted one.

The following test is also included in the `User visits /daylog/create` block:

~~~ javascript
describe('Clicks submit button with valid form', () => {
    before((done) => {
        browser.fill('textarea[name="title"]', DAYLOG.title);
        browser.fill('input[name="slug"]', DAYLOG.slug);
        browser.fill('input[name="location"]', DAYLOG.location);
        browser.fill('input[name="logAt"]', DAYLOG.logAt);
        browser.choose('input[name="category"]', DAYLOG.category);
        browser.pressButton('button[type="submit"]', done);
    });

    it('should show the DayLog view page', () => {
        browser.assert.url(BASE_URL + '/daylog/' + DAYLOG.slug);
    });

    it('should show the view form with the submitted values', () => {
        browser.assert.text('textarea[name="title"]', DAYLOG.title);
        browser.assert.text('input[name="slug"]', DAYLOG.slug);
        browser.assert.text('input[name="location"]', DAYLOG.location);
        browser.assert.text('input[name="logAt"]', DAYLOG.logAt);
        browser.assert.element('input[name="category"][value="' + DAYLOG.category
            + '"][selected="selected"]');
    });

    it('should be in the DayLog home page table', () => {
        browser.clickLink('.card-footer .btn');
        browser.assert.success();
        browser.assert.text('.card-header', 'Day Log');
        browser.assert.text('table', DAYLOG.title);
    });
});
~~~

Another check is the test case `should be in the DayLog home page table` which just returns to the Day Log home page to confirm if the just-created `DayLog` is present in the table.

The `describe` block should fail and reach the timeout because the controller method to catch valid `DayLog`s is still unimplemented.

#### Make the test pass

Add the following in the `then` block of `DayLogController.store`:

~~~ javascript
DayLogModel.create(newDayLog,
    (err, daylog) => {
        if(err)  return res.send("There was a problem adding the information to the database: " + err);

        console.log('Created: "' + daylog.title + '"');

        return res.format({
            html: () => {
                res.redirect("/daylog/" + daylog.slug);
            },
            json: () => { res.json(daylog); }
        }
    );
});
~~~

As mentioned before, `create` is a `mongodb` native method to insert an object to the database and it is directly accessed through the model instance.

After a successful creation, the user will be redirected to the page that shows the information about the just-created `DayLog`. This path `/daylog/:slug` will be the route to the next controller method `show`.

If we run the tests at this point (make sure to re-run the application to apply the changes), then it should fail because the page for `/daylog/:slug` is not implemented yet.

##### Show the `DayLog` information page

The information page will just re-use the form but has disabled fields since it does not require any other detail to expose to the user.

Add its route:

~~~ javascript
router.get('/:slug', DayLogController.show);
~~~

The controller method:

~~~ javascript
show: (req, res) => {
    DayLogModel.findOne({ "slug": req.url.split("/")[2] },
        (err, daylog) => {
            if(err) console.error('GET Error: There was a problem retrieving: ' + err);

            console.info('Show: "' + daylog.title + '"');

            res.format({
                html: () => {
                    res.render('daylog/form', Object.assign({
                            daylog: daylog
                        }, getFormPageConfig(constants.FORM_OPERATIONS.view)));
                },
                json: () => { res.json(daylog); }
            });
        }
    );
}
~~~

The path needs to be split in order to get the slug. For example `/lorem-ipsum`, when split by the backslash, will return the array

    ["", "lorem-ipsum"]

and it shows the slug to be in the 1st index.

Next update the `getFormPageConfig` method's `switch` block with the new feature:

~~~ javascript
case constants.FORM_OPERATIONS.view:
    config.action = null;
    config.method = null;
    break;
~~~

and the constants file:

~~~ javascript
FORM_OPERATIONS: {
    create: 'Create',
    view: 'View'
},
~~~

###### Update the views

The form also needs to be updated to show the information, have the `readonly` or `disabled` properties (depends on the field) and hide the submit and reset buttons since they are not needed.

*   For showing the values in the fields, only show them if the `daylog` object from the controller is not null:

    ~~~ html
    <%= (typeof daylog !== 'undefined') ? daylog.title : '' %>
    ~~~

    This is applicable to the text-based fields but we have radio buttons for the "Category" that needs to be selected as well. The condition is still the same but we add another level to check if the `DayLog`'s category is the same as the radio button's `value`.

    ~~~ html
    <%= (typeof daylog !== 'undefined') ? (daylog.category === "Major" ? 'checked' : '') : '' %>
    ~~~

*   The same pattern applies to the view-only setting of disabling the modification of the fields. Text-based fields are provided with `readonly` and radio buttons with `disabled` respectively:

    ~~~ html
    <%= (operation === constants.FORM_OPERATIONS.view) ? 'readonly' : '' %>
    ~~~

    ~~~ html
    <%= (operation === constants.FORM_OPERATIONS.view) ? 'disabled' : '' %>
    ~~~

*   Finally a simple `if` to not show the form buttons:

    ~~~ html
    <% if(operation !== constants.FORM_OPERATIONS.view) { %>
    <div class="form-group row">
        <div class="offset-sm-2 col-sm-10">
            <button type="submit" class="btn btn-primary">Submit</button>
            <button type="reset" class="btn btn-danger">Reset</button>
        </div>
    </div>
    <% } %>
    ~~~

![DayLog View page](/assets/images/posts/2017-04-29-day-log-app-nodejs-part-3/daylog-view.png){:class="img-responsive"}

Additionally, the Day Log home page table has a incomplete link for each `daylog`. Update the `href` attribute of the cell that shows the `title` from `/<%= daylog.slug %>` to `/daylog/<%= daylog.slug %>`.

#### Run the tests

~~~ javascript
  User visits /daylog
    ✓ should show an alert when no DayLogs are available
    Insert one Day Log in database
      ✓ should show the Day Log in a table when it is added
    Clicks "Create Day Log" button
      ✓ should go to /daylog/create

  User visits /daylog/create
    Clicks submit button with empty form
      ✓ should show 5 "required" error messages near the 5 fields
    Clicks submit button with valid form
      ✓ should show the DayLog view page
      ✓ should show the view form with the submitted values
      ✓ should be in the DayLog home page table


  7 passing (9s)
~~~

### Miscellaneous

#### Put HTML5 `required` attribute

The native Express validation is useful back-end implementation and it is better to take advantage of the available [front-end validation](https://developer.mozilla.org/en/docs/Web/HTML/Element/input#Global_<input>_attributes). Just add `required` in the fields' blocks.

#### Slug middleware validation

Currently there is no way to check the validity of a slug. If we try to go to `/daylog/invalid-slug`, for example and it should not exist in your database, it will stop the application. The browser:

![DayLog invalid slug page - uncatched](/assets/images/posts/2017-04-29-day-log-app-nodejs-part-3/daylog-invalid-slug-uncatched.png){:class="img-responsive"}

The application will throw an error with the following logs:

~~~ console
events.js:160
      throw er; // Unhandled 'error' event
      ^

TypeError: Cannot read property 'title' of null
    at DayLogModel.findOne (/path/to/daylogapp-node/controller/daylog.controller.js:67:48)
    at Query.<anonymous> (/path/to/daylogapp-node/node_modules/mongoose/lib/model.js:3720:16)
    at /path/to/daylogapp-node/node_modules/kareem/index.js:273:21
    at /path/to/daylogapp-node/node_modules/kareem/index.js:127:16
    at _combinedTickCallback (internal/process/next_tick.js:67:7)
    at process._tickCallback (internal/process/next_tick.js:98:9)

npm ERR! Darwin 16.5.0
npm ERR! argv "/usr/local/bin/node" "/usr/local/bin/npm" "start"
npm ERR! node v6.4.0
npm ERR! npm  v3.10.3
npm ERR! code ELIFECYCLE
npm ERR! daylogapp-node@0.0.0 start: `node ./bin/www`
npm ERR! Exit status 1
npm ERR!
npm ERR! Failed at the daylogapp-node@0.0.0 start script 'node ./bin/www'.
npm ERR! Make sure you have the latest version of node.js and npm installed.
npm ERR! If you do, this is most likely a problem with the daylogapp-node package,
npm ERR! not with npm itself.
npm ERR! Tell the author that this fails on your system:
npm ERR!     node ./bin/www
npm ERR! You can get information on how to open an issue for this project with:
npm ERR!     npm bugs daylogapp-node
npm ERR! Or if that isn't available, you can get their info via:
npm ERR!     npm owner ls daylogapp-node
npm ERR! There is likely additional logging output above.

npm ERR! Please include the following file with any support request:
npm ERR!     /path/to/daylogapp-node/npm-debug.log
~~~

In order to solve this, a "middleware" and validator for incoming slugs must be implemented. I decided to put the method as part of a custom Express validator in the controller.

First add the following in the routes:

~~~ javascript
router.use(expressValidator(DayLogController.getCustomExpressValidators));

router.param('slug', DayLogController.getCustomExpressValidators.customValidators.isSlugValid);
~~~

The custom validator must be registered to the `express-validator` instance. The `isSlugValid` will be the checker for a slug's validity and what to do otherwise.

~~~ javascript
getCustomExpressValidators: {
    /**
     * @source http://stackoverflow.com/a/42611431
     */
    customValidators: {
        isSlugValid(req, res, next, slug) {
            DayLogModel.findOne({ "slug": slug },
                (err, daylog) => {
                    if (err || daylog === null) {
                        console.error('DayLog with slug <' + slug + '> was not found');
                        res.status(404)
                        var err = new Error('Not Found');
                        err.status = 404;

                        res.format({
                            html: () => { next(err); },
                            json: () => { res.json({message : err.status  + ' ' + err}); }
                        });
                    } else {
                        req.slug = slug;
                        next();
                    }
                }
            )
        }
    }
},
~~~

Now if we revisit the invalid slug, the application will not stop:

~~~ console
DayLog with slug <invalid-slug> was not found
GET /daylog/invalid-slug 404 37.267 ms - 1897
~~~

and will show the following page:

![DayLog invalid slug page - uncatched](/assets/images/posts/2017-04-29-day-log-app-nodejs-part-3/daylog-invalid-slug-catched.png){:class="img-responsive"}

It is obvious that the error page needs more work but I will leave it to future improvements such as providing a better description and a link back to the create page.

#### Implement other validation requirements

The requirement for a slug is its uniqueness. However Express validation has no native assertion that checks the database; fortunately we can create custom validator as demonstrated in `isSlugValid`.

Just add the new `customValidator` called `isSlugUnique` that takes a slug and an operation:

~~~ javascript
isSlugUnique(slug, operation) {
    return new Promise((resolve, reject) => {
        DayLogModel.findOne({ slug: slug }, (err, daylog) => {
            if(err) throw err;

            if(daylog === null) {
                resolve();
            } else if(slug === daylog.slug &&
                operation === constants.FORM_OPERATIONS.edit) {
                resolve();
            } else {
                reject();
            }
        })
    })
}
~~~

The basic flow is if the `findOne` method does not find any `DayLog` with the same slug, it passes with `resolve` and `reject` otherwise. However we have a special case for editing - the `DayLog` already exists with the same slug so we need to override it with a `resolve` as well.

Add the new `FORM_OPERATIONS.edit` in the constans file:

~~~ javascript
edit: 'Edit'
~~~

In order to use it alongside the other Express assertions, just call it as any other constraint after the `assert`:

~~~ javascript
request.assert('slug', 'Slug is already in use').isSlugUnique(operation);
~~~

![DayLog invalid slug page - uncatched](/assets/images/posts/2017-04-29-day-log-app-nodejs-part-3/daylog-form-error-slug-nonunique.png){:class="img-responsive"}

## References
* Alam, Zubair. "How to include a template with parameters in EJS?" *Stack Overflow*. N.p., 17 Nov. 2016. Web. 28 Apr. 2017. <[`http://stackoverflow.com/a/40652080`](http://stackoverflow.com/a/40652080)>.
* Benihana21. "How to use zombie to test ." *Javascript - How to use zombie to test - Stack Overflow*. N.p., 6 June 2015. Web. 28 Apr. 2017. <[`http://stackoverflow.com/a/30686579`](http://stackoverflow.com/a/30686579)>.
* Coleman, Kendrick. "How to Create a Complete Express.js Node.js MongoDB CRUD and REST Skeleton." *Airpair*. N.p., 2015. Web. 20 Apr. 2017. <[`https://www.airpair.com/javascript/complete-expressjs-nodejs-mongodb-crud-skeleton`](https://www.airpair.com/javascript/complete-expressjs-nodejs-mongodb-crud-skeleton)>.
* J, Peter. "How can I know which radio button is selected via jQuery?" *Javascript - How can I know which radio button is selected via jQuery? - Stack Overflow*. N.p., 27 Feb. 2009. Web. 28 Apr. 2017. <[`http://stackoverflow.com/a/596369`](http://stackoverflow.com/a/596369)>.
* Make Tips. "Validation In Express-Validator." *Node.js - Validation In Express-Validator - Stack Overflow*. N.p., 5 Mar. 2017. Web. 20 Apr. 2017. <[`http://stackoverflow.com/a/42611431`](http://stackoverflow.com/a/42611431)>.
* Prestifilippo, Matthew. "How does assert (req.assert) work in nodejs." *Javascript - How does assert (req.assert) work in nodejs - Stack Overflow*. N.p., 5 May 2014. Web. 28 Apr. 2017. <[`http://stackoverflow.com/a/23473515`](http://stackoverflow.com/a/23473515)>.