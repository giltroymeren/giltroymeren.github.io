---
layout: post
title:  "Day Log App in Node.js Part 8: Add user authentication"
date:   2017-05-16
categories: projects notes node.js series daylogapp authentication passport
---

I wanted to learn how to implement user authentication in a Node application. In our [first Laravel implementation]({{ site.baseurl }}{% post_url 2017-03-20-day-log-app %}), it was an easy task because this feature is already provided at the start with just a command. However Node requires external libraries for such functionality.

Fortunately I found [Daniel Gynn's easy-to-follow tutorial](https://www.danielgynn.com/build-an-authentication-app-using-express-node-passport/) on how to integrate the [Passport](http://passportjs.org/) library into our current application.

## Table of Contents
* TOC
{:toc}

## Steps

The following steps and implementation were mostly taken from Gynn's original article.

### Install dependencies

The following packages are needed:

* [passport](http://passportjs.org/)
* [passport-local](https://github.com/jaredhanson/passport-local)
* [bcrypt-nodejs](https://www.npmjs.com/package/bcrypt-nodejs)
* [connect-flash](https://github.com/jaredhanson/connect-flash)
* [express-session](https://github.com/expressjs/session)

Install them with the command:

    npm install passport passport-local bcrypt-nodejs connect-flash express-session --save

### Setup dependencies

The packages are setup in `app.js`:

~~~ javascript
var passport = require('passport');
var LocalStrategy = require('passport-local').Strategy;
var flash = require('connect-flash');
var session = require('express-session');
...
app.use(session({ secret: 'shhsecret' }));
app.use(passport.initialize());
app.use(passport.session());
app.use(flash());
~~~

### `User` model

Create the file `model/user.model.js` with just the username and password fields.

~~~ javascript
'use strict';

var mongoose = require('mongoose');
var bcrypt = require('bcrypt-nodejs');

var UserSchema = new mongoose.Schema({
    username: {
        type: String,
        required: true,
        unique: true
    },
    password: {
        type: String,
        required: true
    }
});

UserSchema.methods.generateHash = function(password) {
    return bcrypt.hashSync(password, bcrypt.genSaltSync(8), null);
};

UserSchema.methods.validPassword = function(password) {
    return bcrypt.compareSync(password, this.password);
};

module.exports = mongoose.model('User', UserSchema);
~~~

The methods `generateHash` and `validPassword` will be used later for the authentication process.

### Routes

Create the `routes/user.route.js` file:

~~~ javascript
'use strict';

var express = require('express');
var router = express.Router();
var database = require('../config/database');

var UserController = require('../controller/user.controller');

router.get('/', UserController.index);

router.get('/register', UserController.showRegister);

router.post('/register', UserController.register);

router.get('/login', UserController.showLogin);

router.post('/login', UserController.login);

router.get('/logout', UserController.logout);

module.exports = router;
~~~

This will manage the routes to login, registration and logout and their implementations are in the controller file.

### Controller

Create the file `controller/user.controller.js`:

~~~ javascript
'use strict';

var passport = require('passport');

var UserModel = require('../model/user.model');
var constants = require('../config/constants');

const LOGIN_URL = "/user/login";
const REGISTER_URL = "/user/register";
const FORM_VIEW = "user/form";

var UserController = {
    index: (request, response) => {
        response.render('index');
    },

    showRegister: (request, response) => {
        if(request.isAuthenticated()) {
            response.redirect('/');
        } else {
            response.render(FORM_VIEW, {
                message: request.flash('error-register'),
                type: "alert-danger",
                operation: "Register",
                action: REGISTER_URL
            });
        }
    },

    register: passport.authenticate('register', {
        successRedirect: '/',
        failureRedirect: REGISTER_URL,
        operation: "Register",
        action: REGISTER_URL,
        failureFlash: true,
    }),

    showLogin: (request, response) => {
        if(request.isAuthenticated()) {
            response.redirect('/');
        } else {
            response.render(FORM_VIEW, {
                message: request.flash('error-login'),
                type: "alert-danger",
                operation: "Login",
                action: LOGIN_URL
            });
        }
    },

    login: passport.authenticate('login', {
        successRedirect: '/',
        failureRedirect: LOGIN_URL,
        failureFlash: true,
    }),

    logout: (request, response) => {
        request.logout();
        response.redirect('/');
    }
};

module.exports = UserController;
~~~

The `index` method points to the application's main page.

`showRegister` and `showLogin` shows the form with their respective attributes. `register` and `login` uses Passport's `authenticate` method to handle the respective logic specified by the first parameters (namely "register" and "login") which are defined in `config/passport.js`.

Logout simply logs out the user and returns to the index page.

### Form page

Both the registration and login uses the same form with different attributes to differentiate the two.

~~~ html
<% include ../partials/header %>

<div class="card">
    <h3 class="card-header"><%= operation %></h3>

    <div class="card-block">

        <form action="<%= action %>" method="post">

            <% if (typeof message !== 'undefined') { %>
                <% if (message.length) { %>
                    <%- include('../partials/alert', { message: message, type: type }) %>
            <% }
            } %>

            <div class="container">
                <div class="form-group row">
                    <label for="username" class="col-sm-2 col-form-label text-right">Username</label>
                    <div class="col-sm-10">
                        <input type="text" class="form-control" id="username" name="username"
                            placeholder="Enter your username" required>
                    </div>
                </div>

                <div class="form-group row">
                    <label for="password" class="col-sm-2 col-form-label text-right">Password</label>
                    <div class="col-sm-10">
                        <input type="password" class="form-control" id="password" name="password"
                            placeholder="Enter your password" required>
                    </div>
                </div>

                <div class="form-group row">
                    <div class="offset-sm-2 col-sm-10">
                        <button type="submit" class="btn btn-primary">Login</button>
                        <button type="reset" class="btn btn-danger">Reset</button>
                    </div>
                </div>
            </div><!-- .container -->

        </form>

    </div><!-- .card-block -->

    <div class="card-footer text-center">
        <a href="/" class=" btn btn-primary btn-sm">Back</a>
    </div>

</div><!-- .card -->


<% include ../partials/footer %>
~~~

![Login page](/assets/images/posts/2017-05-16-day-log-app-nodejs-part-8/page-register.png){:class="img-responsive"}
![Register page](/assets/images/posts/2017-05-16-day-log-app-nodejs-part-8/page-login.png){:class="img-responsive"}

Authentication messages are shown when there is a failure that requires modification in `partials/alert.ejs`:

~~~ javascript
<% if(typeof errors !== "undefined") { %>
    <%= errors.find(error => error.param === field).msg %>
<% } else { %>
    <%= message %>
<% } %>
~~~

### Header options

I also made changes to the header file to show the user options of "Login", "Registration", and "Logout" depending on the session.

~~~ html
<nav class="navbar navbar-toggleable-md navbar-light bg-faded"
    style="margin-bottom: 2em">
    <div class="container col-sm-6">
        <button class="navbar-toggler navbar-toggler-right" type="button"
            data-toggle="collapse" data-target="#navbar"
            aria-controls="navbar" aria-expanded="false" aria-label="Toggle navigation">
            <span class="navbar-toggler-icon"></span>
        </button>
        <a class="navbar-brand" href="/">DayLogApp</a>

        <div class="collapse navbar-collapse" id="navbar">
            <ul class="nav navbar-nav ml-auto w-100 justify-content-end">
                <% if(typeof loggedIn == 'undefined') { %>
                    <li class="nav-item">
                        <a class="nav-link" href="/user/login">Login</a>
                    </li>
                    <li class="nav-item">
                        <a class="nav-link" href="/user/register">Register</a>
                    </li>
                <% } else if(loggedIn) { %>
                    <li class="nav-item">
                        <a class="nav-link" href="/user/logout">Logout</a>
                    </li>
                <% } %>
            </ul>
        </div>
    </div>
</nav>
~~~

![Header register and login](/assets/images/posts/2017-05-16-day-log-app-nodejs-part-8/header-default.png){:class="img-responsive"}
![Header logout](/assets/images/posts/2017-05-16-day-log-app-nodejs-part-8/header-logout.png){:class="img-responsive"}

### Passport configuration

Create the file `config/passport.js`:

~~~ javascript
var LocalStrategy = require('passport-local').Strategy;
var UserModel = require('../model/user.model');

const localStrategy = {
    usernameField: 'username',
    passwordField: 'password',
    passReqToCallback: true,
};

module.exports = (passport) => {
    passport.serializeUser((user, done) => {
        done(null, user.id);
    });

    passport.deserializeUser((id, done) => {
        UserModel.findById(id, (error, user) => {
            done(error, user);
        });
    });

    passport.use('register',
        new LocalStrategy(localStrategy,
            (request, username, password, done) => {
                process.nextTick(() => {
                    UserModel.findOne({ username:  username },
                        (error, user) => {
                            if (error) return done(error);

                            if (user) {
                                return done(null, false,
                                    request.flash('error-register',
                                        'Username "' + username + '" is already in use.'));
                            } else {
                                var newUser = new UserModel();
                                newUser.username = username;
                                newUser.password = newUser.generateHash(password);
                                newUser.save((error) => {
                                    if (error) throw error;

                                    return done(null, newUser);
                                });
                            }
                        }
                    );
                });
        }
    ));

    passport.use('login',
        new LocalStrategy(localStrategy,
            (request, username, password, done) => {
                UserModel.findOne({ username:  username },
                    (error, user) => {
                        if (error) throw error;

                        if (!user)  {
                            return done(null, false,
                                request.flash('error-login',
                                    'No username "' + username + '" found.'));
                        }

                        if (!user.validPassword(password)) {
                            return done(null, false,
                                request.flash('error-login',
                                    'Wrong username/password combination.'));
                        }

                        return done(null, user);
                    }
                );
            }
        )
    );
};
~~~

The Passport `use` declarations "register" and "login" correspond to the same names in the controller with the `authenticate` method. Inside each are flash messages that are also called in the controller.

Add this in `app.js` as a dependency to use:

~~~ javascript
require('./config/passport')(passport);
~~~

### User page restrictions

The `DayLog` page is only accessible to logged-in users so a few changes are reguired.

In `routes/index.route.js`, we set the variable `loggedIn` to `true` if the user is authenticated:

~~~ javascript
router.use((req, res, next) => {
    if (req.isAuthenticated()) res.locals.loggedIn = true;

    return next();
});
~~~

This `loggedIn` variable is the one checked in the header view file to choose which user option to show.

Next, we add a new method in `controller/daylog.controller.js` to redirect all un-logged-in users that want to access the `/daylog/*` route. Here are the changes in `route/daylog.route.js` and the controller respecitvely:

~~~ javascript
router.use(DayLogController.isUserLoggedIn);
~~~

~~~ javascript
isUserLoggedIn: (request, response, next) => {
    if (!request.isAuthenticated()) {
        response.redirect('/user/login');
    } else {
        return next();
    }
},
~~~

## Problems

I encountered about an interesting problem during development. I made the methods in the Passport configuration file to use the arrow signature:

~~~ javascript
UserSchema.methods.validPassword = (password) => {
    return bcrypt.compareSync(password, this.password);
};
~~~

When I tried to login with OK details (username and password present in database) (I found no problems when the credentials are invalid), the application stopped with the following error log:

~~~ console
events.js:165
      throw err;
      ^

Error: Uncaught, unspecified "error" event. (Incorrect arguments)
    at Function.emit (events.js:163:17)
    at Query.<anonymous> (/path/to/daylogapp-node/node_modules/mongoose/lib/model.js:3739:13)
    at /path/to/daylogapp-node/node_modules/kareem/index.js:277:21
    at /path/to/daylogapp-node/node_modules/kareem/index.js:131:16
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

I learned that [accessing `this` in arrow-style methods](http://stackoverflow.com/a/41374913) is the cause of this.

## References
* "4.x API." *Express 4.x - API Reference*. N.p., n.d. Web. 13 May 2017. <[`http://expressjs.com/en/4x/api.html#res.locals`](http://expressjs.com/en/4x/api.html#res.locals)>.
* Da Silva, João Rocha. "NodeJS (Express.js) : How to make the Session global." *Node.js - NodeJS (Express.js) : How to make the Session global - Stack Overflow*. N.p., 20 Apr. 2017. Web. 13 May 2017. <[`http://stackoverflow.com/a/43526061`](http://stackoverflow.com/a/43526061)>.
* Gynn, Daniel. "Authenticating Users in a Node App using Express & Passport (Part One)." *Daniel Gynn*. Daniel Gynn, 22 Nov. 2015. Web. 7 May 2017. <[`https://www.danielgynn.com/build-an-authentication-app-using-express-node-passport/`](https://www.danielgynn.com/build-an-authentication-app-using-express-node-passport/)>.
* Herman, Michael. "Michael Herman." *User Authentication With Passport and Express 4 - Michael Herman*. N.p., 31 Jan. 2015. Web. 12 May 2017. <[`http://mherman.org/blog/2015/01/31/local-authentication-with-passport-and-express-4/`](http://mherman.org/blog/2015/01/31/local-authentication-with-passport-and-express-4/)>.
* Klep, Robert. "Express Passport (node.js) error handling." *Stack Overflow*. N.p., 29 Mar. 2013. Web. 15 May 2017. <[`http://stackoverflow.com/a/15711502`](http://stackoverflow.com/a/15711502)>.
* Mihejevs, Maksims. "How to know if user is logged in with passport.js?" *Node.js - How to know if user is logged in with passport.js? - Stack Overflow*. N.p., 11 Sept. 2013. Web. 13 May 2017. <[`http://stackoverflow.com/a/18739922`](http://stackoverflow.com/a/18739922)>.
* Torda, Francis. "Uncaught, unspecified "error" event." *Node.js - Uncaught, unspecified "error" event - Stack Overflow*. N.p., 29 Dec. 2016. Web. 13 May 2017. <[`http://stackoverflow.com/a/41374913`](http://stackoverflow.com/a/41374913)>.
* Yashua. "How to know if user is logged in with passport.js?" *Node.js - How to know if user is logged in with passport.js? - Stack Overflow*. N.p., 18 Nov. 2013. Web. 13 May 2017. <[`http://stackoverflow.com/a/20056529`](http://stackoverflow.com/a/20056529)>.
* Zeiss, Mirco. "Working with Sessions in Express.js." *Node.js - Working with Sessions in Express.js - Stack Overflow*. N.p., 8 Jan. 2013. Web. 13 May 2017. <[`http://stackoverflow.com/questions/14218725/working-with-sessions-in-express-js`](http://stackoverflow.com/questions/14218725/working-with-sessions-in-express-js)>.