---
layout: post
title:  "Day Log App in Node.js Part 9: User-DayLog ownership"
date:   2017-05-17
categories: projects notes node.js series daylogapp
---

[User authentication]({{ site.baseurl }}{% post_url 2017-05-16-day-log-app-nodejs-part-8 %}) has been implemented in the previous post. Currently all users who are logged-in can access all Day Logs made by other users.

This post will show how to implement a one-to-many relationship between `User` and `DayLog` objects and to limit access to Day Logs created by the logged-in user.

## Table of Contents
* TOC
{:toc}

## Steps

### Update model

A `DayLog` object has a knowledge of its owner, a `User`. In `model/daylog.model.js` a new attribute is added: `user` which corresponds to the `User` model.

~~~ javascript
var Schema = mongoose.Schema;
var ObjectId = Schema.ObjectId;

var UserModel = require('./user.model');

...

    user: {
        type: ObjectId,
        ref: 'UserModel'
    }
~~~

Through this, a `DayLog` is created, a `User` model can just be assigned to the `user` attribute. No need to provide the actual `id` of the `User` object.

### Show created `DayLog`s

All `DayLog`s are sent to the home page in the `DayLogController.index` method. Just add the new filter to search for the ones by the logged-in user.

~~~ javascript
DayLogModel.find({ user: request.user },
~~~

### Prevent access to other `DayLogs`

All `DayLog`s are accessible through their `slug` attribute. Even though only the user-created ones are shown in the home page, there is also a way to access the others through the URL.

Fortunately the methods to view, edit and delete the objects have the same signature as `index` which means adding the `user` filter in `find` will either return a valid result or `null` if none matched the criteria.

`DayLogController` methods `edit`, `update` and `delete` will have this new check in their `findOne` methods:

~~~ javascript
DayLogModel.findOne({ slug: newDayLog.slug, user: req.user },
~~~

The result is checked inside the callback. If `null`, then redirect to the home page with a warning about invalid access else to continue with the command.

~~~ javascript
if(daylog === null) return showHomeIfNonUserAccessesDayLog(request, response, "edit");
~~~

The third parameter is just to customize the message.

The method `showHomeIfNonUserAccessesDayLog` also needs the [`express-flash`](https://github.com/RGBboy/express-flash) package to enable sending of messages or variable with a `redirect`.

~~~ javascript
function showHomeIfNonUserAccessesDayLog(request, response, method) {
    var message = "Cannot " + method + " daylog not under your account.";
    console.log(message);
    request.flash('message', message);
    request.flash('type', 'alert-warning');
    response.redirect('/daylog');
}
~~~

In addition, just setting these flash message here will not work right away. Calling the URL `/daylog` will actually point to the `index` method, which has its own attribute setting for the view file. The flash messages need to be fetched again this time and re-set.

~~~ javascript
response.render('daylog/index', {
    daylogs: daylogs,
    message: request.flash('message'),
    type: request.flash('type')
});
~~~

Lastly add the message template in `views/daylog/index.ejs`:

~~~ html
<% if (typeof message !== 'undefined') { %>
    <% if (message.length) { %>
        <%- include('../partials/alert', { message: message, type: type }) %>
<% }
} %>
~~~

## References
* Dan. "Node.js - Creating Relationships with Mongoose." *Javascript - Node.js - Creating Relationships with Mongoose - Stack Overflow*. N.p., 18 Oct. 2011. Web. 16 May 2017. <[`http://stackoverflow.com/a/7813331`](http://stackoverflow.com/a/7813331)>.