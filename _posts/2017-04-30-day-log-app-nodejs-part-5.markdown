---
layout: post
title:  "Day Log App in Node.js Part 5: Edit a DayLog"
date:   2017-04-30
categories: projects notes node.js express series daylogapp
---

This post will discuss how to implement the page to edit a DayLog.

## Table of Contents
* TOC
{:toc}

## Development

Each `DayLog` is editable and this should be accessible from the home page with the "Edit" button in the table.

![DayLog index page - not empty table](/assets/images/posts/2017-04-27-day-log-app-nodejs-part-2/daylog-table.png){:class="img-responsive"}

> **NOTE**: I decided to exclude the BDD testing in this part because I am having problems in implementing the edit test cases.

### Update the link

In `/views/daylog/index.ejs`. update the link path of the "Edit" button from:

~~~ html
<a href="/<%= daylog.slug %>/edit" class="btn btn-primary btn-sm">Edit</a>
~~~

to the pattern `/daylog/:slug/edit`:

~~~ html
<a href="/daylog/<%= daylog.slug %>/edit" class="btn btn-primary btn-sm">Edit</a>
~~~

### Add the route

> **NOTE**: Make sure that there's an exisiting `DayLog` in the database.

If we click the "Edit" button right now it should return a `404` so this route should be added:

~~~ javascript
router.get('/:slug/edit', DayLogController.edit);
~~~

This should recognize the pattern `/daylog/:slug/edit` as intended.

### Show the "Edit" form

Add the method `edit` after `show`:

~~~ javascript
edit: (req, res) => {
    DayLogModel.findOne({ "slug": req.url.split("/")[1] },
        (err, daylog) => {
            if(err) return console.error('GET Error: There was a problem retrieving: ' + err);

            res.format({
                html: () => {
                    res.render('daylog/form', Object.assign({
                            daylog : daylog
                        }, getFormPageConfig(constants.FORM_OPERATIONS.edit, daylog))
                    );
                },
                json: () => { res.json(daylog); }
            });
        }
    );
}
~~~

and a new `switch` case in `getFormPageConfig`:

~~~ javascript
case constants.FORM_OPERATIONS.edit:
    config.action = '/daylog/' + daylog.slug + '/edit';
    break;
~~~

This `action` path will be the same but a `POST` HTTP method.

### Disabled `slug` modification

The `slug` attribute is unique and I decided to disable its modification after creation.

Update the `readonly` conditional from:

~~~ html
<%= (operation === constants.FORM_OPERATIONS.view) ? 'readonly' : '' %>
~~~

to:

~~~ html
<%= (operation === constants.FORM_OPERATIONS.view ||
    operation === constants.FORM_OPERATIONS.edit) ? 'readonly' : '' %>
~~~

![DayLog Edit page](/assets/images/posts/2017-04-30-day-log-app-nodejs-part-5/daylog-edit.png){:class="img-responsive"}

If we re-run the application and submit the edit form, then a `404` page should appear again.

### Submit the "Edit" form

Add the `PUT` route with the same URL pattern:

~~~ javascript
router.put('/:slug/edit', DayLogController.update);
~~~

Implement the controller method `update`:

~~~ javascript
update: (req, res) => {
    validateDayLog(req, constants.FORM_OPERATIONS.edit);
    var newDayLog = getDayLogFromRequest(req);

    req.asyncValidationErrors()
        .then(() => {
            DayLogModel.findOne({ "slug": newDayLog.slug },
                (err, daylog) => {
                    daylog.update(newDayLog, (err, slug) => {
                        if(err) res.send("There was a problem updating the information to the database: " + err);

                        console.info('Editing: "' + daylog.title + '"');

                        res.format({
                            html: () => { res.redirect("/daylog/" + daylog.slug); },
                            json: () => { res.json(daylog); }
                        });
                    })
                }
            );
        })
        .catch((errors) => {
            return res.format({
                html: () => {
                    res.render('daylogs/form', Object.assign({
                            daylog: newDayLog,
                            errors: errors
                        }, getFormPageConfig(constants.FORM_OPERATIONS.edit, newDayLog))
                    );
                }
            });
        });
}
~~~

When the update is successful, the page should be redirected to the view page with the new information or back in the edit form with validation errors.

### Override the requests

In addition, since this route is `PUT`, we need to add a hidden field in the form to be set during the edit mode:

~~~ html
<% if(operation === constants.FORM_OPERATIONS.edit) { %>
    <input type="hidden" value="PUT" name="_method">
<% } %>
~~~

Finally the library ["method override"](https://github.com/expressjs/method-override) needs to be added as well:

    npm install method-override --save

and setup in `app.js`:

~~~ javascript
var methodOverride = require('method-override');
...
app.use(methodOverride((req, res) => {
    if (req.body && typeof req.body === 'object' && '_method' in req.body) {
        var method = req.body._method
        delete req.body._method
        return method
    }
}));
~~~

### `DayLog` ID changed

Once the steps above have been set, if we try to edit a `DayLog`, it should show a page with the following message:

> There was a problem updating the information to the database: MongoError: After applying the update to the document {_id: ObjectId(\<old-id\>) , ...}, the (immutable) field '_id' was found to have been altered to _id: ObjectId(\<new-id\>)

and stop the program with the error logs:

~~~ console
GET /daylog/lorem-ipsum/edit 200 64.349 ms - 6424
Editing: "Lorem ipsum ipsum dolor sit amet"
_http_outgoing.js:344
    throw new Error('Can\'t set headers after they are sent.');
    ^

Error: Can't set headers after they are sent.
    at ServerResponse.OutgoingMessage.setHeader (_http_outgoing.js:344:11)
    at vary (/path/to/daylogapp-node/node_modules/vary/index.js:129:9)
    at ServerResponse.res.vary (/path/to/daylogapp-node/node_modules/express/lib/response.js:925:3)
    at ServerResponse.res.format (/path/to/daylogapp-node/node_modules/express/lib/response.js:630:8)
    at daylog.update (/path/to/daylogapp-node/controller/daylog.controller.js:155:33)
    at callback (/path/to/daylogapp-node/node_modules/mongoose/lib/query.js:2612:9)
    at /path/to/daylogapp-node/node_modules/kareem/index.js:216:48
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

It is clear from the `mongodb` message the a new ID has been set to the existing and this was caused by the creation of a new `DayLog` in the controller method `getDayLogFromRequest()`.

To solve this, just remove `new DayLogModel(..)` and just pass an `Object`:

~~~ javascript
return {
    title : request.body.title,
    slug : request.body.slug,
    location : request.body.location,
    logAt : (request.body.logAt) ? new Date(request.body.logAt) : '',
    category : request.body.category
};
~~~

The edit feature should not have any other errors when submitted whether the form has been modified or not; the changes should be seen in both the home table (title and date updated) and view page.

## References
* Coleman, Kendrick. "How to Create a Complete Express.js Node.js MongoDB CRUD and REST Skeleton." *Airpair*. N.p., 2015. Web. 20 Apr. 2017. <[`https://www.airpair.com/javascript/complete-expressjs-nodejs-mongodb-crud-skeleton`](https://www.airpair.com/javascript/complete-expressjs-nodejs-mongodb-crud-skeleton)>.
* Liew, Zell. "Building a Simple CRUD Application with Express and MongoDB." *Zell Liew*. N.p., 22 Jan. 2016. Web. 20 Apr. 2017. <[`https://zellwk.com/blog/crud-express-mongodb/`](https://zellwk.com/blog/crud-express-mongodb/)>.
* Mangela, Siddhesh. "Siddacool/airpair-crud-app." *GitHub*. N.p., 23 Mar. 2016. Web. 20 Apr. 2017. <[`https://github.com/siddacool/airpair-crud-app`](https://github.com/siddacool/airpair-crud-app)>.
* Pérez, Alfonso. "Node.js mongodb .. the (immutable) field '_id' was found to have been altered." *Javascript - node.js mongodb .. the (immutable) field '_id' was found to have been altered - Stack Overflow*. N.p., 14 Sept. 2015. Web. 30 Apr. 2017. <[`http://stackoverflow.com/a/32563554`](http://stackoverflow.com/a/32563554)>.
* Sekar, Raja. "A TDD Approach to Building a Todo API Using Node.js and MongoDB." *Semaphore - Continuous Integration, Deployment, TDD, DevOps tutorials*. N.p., 29 June 2016. Web. 20 Apr. 2017. <[`https://semaphoreci.com/community/tutorials/a-tdd-approach-to-building-a-todo-api-using-node-js-and-mongodb`](https://semaphoreci.com/community/tutorials/a-tdd-approach-to-building-a-todo-api-using-node-js-and-mongodb)>.
* Sevilleja, Chris. "Easily Develop Node.js and MongoDB Apps with Mongoose." *Scotch*. N.p., 14 Jan. 2015. Web. 20 Apr. 2017. <[`https://scotch.io/tutorials/using-mongoosejs-in-node-js-and-mongodb-applications`](https://scotch.io/tutorials/using-mongoosejs-in-node-js-and-mongodb-applications)>.
