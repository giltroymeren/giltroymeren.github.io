---
layout: post
title:  "Day Log App in Node.js Part 7: Deploy in Heroku"
date:   2017-05-04
categories: projects notes node.js series daylogapp deployment heroku
---

This article will discuss how to deploy our [node.js](https://nodejs.org/en/) Day Log App to [Heroku](https://www.heroku.com/). There are available instructions online to perform this but I faced certain problems that were not in a singular post so I decided to document my entire flow here.

## Table of Contents
* TOC
{:toc}

## Steps

Most of the steps here were just taken from the official [Heroku Node.js deployment post](https://devcenter.heroku.com/articles/deploying-nodejs). I added the logs (if there were) for reference.

### Optional

I mentioned in a [previous Heroku deployment article]({{site_url}}{% post_url 2017-04-01-how-to-deploy-day-log-app-in-heroku %}#manage-branches) that I use a different branch apart from the `master` to separate Heroku-related configuration changes from the actual code base.

### Declare Node.js version

Heroku requires the version of Node.js where the application was built on. Just type `node -v` in the console (`v6.4.0` in my case).

Add it in the project's `package.json`:

~~~ json
"engines": {
  "node": "6.4.0"
},
~~~

### Set `PORT` to Heroku default

During deployment the application did not run and from the Heroku logs I got the following message:

    Error R10 (Boot timeout) -> Web process failed to bind to $PORT within 60 seconds of launch

To solve, I changed `3000` to `5000`.

### Add `Procfile`

Heroku uses a `Procfile` and for my configuration I used the following:

    worker: node app.js

Unfortunately the suggested form below did not work for me and only returned an error.

    node: web app.js

### Create the application

The format to create a Heroku application in the [CLI](https://devcenter.heroku.com/articles/heroku-cli) is `heroku create`. `-a <app-name>` is added if you do not want Heroku to randomly assign an application name.

~~~ console
$ heroku create -a daylogapp-node
Creating ⬢ daylogapp-node... done
https://daylogapp-node.herokuapp.com/ | https://git.heroku.com/daylogapp-node.git
~~~

### Add `mongolab` add-on

Since this application uses MongoDB and `mongoose`, an add-on for this is needed. I will defer the steps to external sources because it is simple enough to follow.

~~~ console
$  heroku addons:create mongolab
Creating mongolab on ⬢ daylogapp-node... free
Welcome to mLab.  Your new subscription is being created and will be available shortly.  Please consult the mLab Add-on Admin UI to check on its progress.
Created your-new-mongolab-uri as MONGODB_URI
Use heroku addons:docs mongolab to view documentation
~~~

After a successful creation, visit your project's "Overview" page and click the "mLab MongoDB" entry in the "Installed add-ons" section to view the MongoLab page for it.

![Heroku - project page - Overview tab](/assets/images/posts/2017-05-04-day-log-app-nodejs-part-7/heroku-project-overview.png){:class="img-responsive"}

The database URL will be available there and copy it to the definitions in `constants.js`:

~~~ javascript
DATABASE: {
    production: process.env.MONGOLAB_URI ||
                'mongodb://<db-username>:<db-password>@uniqueurl.mlab.com:port/heroku_url' ||
                'mongodb://localhost/daylogapp',
~~~

I added the variable `process.env.MONGOLAB_URI` as suggested in the official guide. Make sure to replace the variables `db-username` and `db-password`.

### Deploy

First, `push` all changes to your repository and use the command `git push heroku master` to start deployment.

~~~ console
$ git push origin deploy
Counting objects: 14, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (12/12), done.
Writing objects: 100% (14/14), 1.41 KiB | 0 bytes/s, done.
Total 14 (delta 7), reused 0 (delta 0)
remote:
remote: Create pull request for deploy:
remote:   https://bitbucket.org/giltroymeren/daylogapp-node/pull-requests/new?source=deploy&t=1
remote:
To https://giltroymeren@bitbucket.org/giltroymeren/daylogapp-node.git
 * [new branch]      deploy -> deploy
Giltroys-MacBook-Pro:daylogapp-node giltroymeren$ git push heroku master
Counting objects: 164, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (138/138), done.
Writing objects: 100% (164/164), 22.22 KiB | 0 bytes/s, done.
Total 164 (delta 59), reused 0 (delta 0)
remote: Compressing source files... done.
remote: Building source:
remote:
remote: -----> Node.js app detected
remote:
remote: -----> Creating runtime environment
remote:
remote:        NPM_CONFIG_LOGLEVEL=error
remote:        NPM_CONFIG_PRODUCTION=true
remote:        NODE_VERBOSE=false
remote:        NODE_ENV=production
remote:        NODE_MODULES_CACHE=true
remote:
remote: -----> Installing binaries
remote:        engines.node (package.json):  unspecified
remote:        engines.npm (package.json):   unspecified (use default)
remote:
remote:        Resolving node version 6.x via semver.io...
remote:        Downloading and installing node 6.10.3...
remote:        Using default npm version: 3.10.10
remote:
remote: -----> Restoring cache
remote:        Skipping cache restore (new runtime signature)
remote:
remote: -----> Building dependencies
remote:        Installing node modules (package.json)
remote:        daylogapp-node@0.0.0 /tmp/build_36e0eb912d737ac3e2bf1feb887cfc49
remote:        ├─┬ body-parser@1.17.1
remote:        │ ├── bytes@2.4.0
remote:        │ ├── content-type@1.0.2
remote:        │ ├── debug@2.6.1
remote:        │ ├── depd@1.1.0
remote:        │ ├─┬ http-errors@1.6.1
remote:        │ │ └── inherits@2.0.3
remote:        │ ├── iconv-lite@0.4.15
remote:        │ ├─┬ on-finished@2.3.0
remote:        │ │ └── ee-first@1.1.1
remote:        │ ├── qs@6.4.0
remote:        │ ├─┬ raw-body@2.2.0
remote:        │ │ └── unpipe@1.0.0
remote:        │ └─┬ type-is@1.6.15
remote:        │   ├── media-typer@0.3.0
remote:        │   └─┬ mime-types@2.1.15
remote:        │     └── mime-db@1.27.0
remote:        ├─┬ cookie-parser@1.4.3
remote:        │ ├── cookie@0.3.1
remote:        │ └── cookie-signature@1.0.6
remote:        ├─┬ debug@2.6.6
remote:        │ └── ms@0.7.3
remote:        ├── ejs@2.5.6
remote:        ├─┬ express@4.15.2
remote:        │ ├─┬ accepts@1.3.3
remote:        │ │ └── negotiator@0.6.1
remote:        │ ├── array-flatten@1.1.1
remote:        │ ├── content-disposition@0.5.2
remote:        │ ├── debug@2.6.1
remote:        │ ├── encodeurl@1.0.1
remote:        │ ├── escape-html@1.0.3
remote:        │ ├── etag@1.8.0
remote:        │ ├─┬ finalhandler@1.0.2
remote:        │ │ └─┬ debug@2.6.4
remote:        │ │   └── ms@0.7.3
remote:        │ ├── fresh@0.5.0
remote:        │ ├── merge-descriptors@1.0.1
remote:        │ ├── methods@1.1.2
remote:        │ ├── parseurl@1.3.1
remote:        │ ├── path-to-regexp@0.1.7
remote:        │ ├─┬ proxy-addr@1.1.4
remote:        │ │ ├── forwarded@0.1.0
remote:        │ │ └── ipaddr.js@1.3.0
remote:        │ ├── range-parser@1.2.0
remote:        │ ├─┬ send@0.15.1
remote:        │ │ ├── debug@2.6.1
remote:        │ │ ├── destroy@1.0.4
remote:        │ │ └── mime@1.3.4
remote:        │ ├── serve-static@1.12.1
remote:        │ ├── setprototypeof@1.0.3
remote:        │ ├── statuses@1.3.1
remote:        │ ├── utils-merge@1.0.0
remote:        │ └── vary@1.1.1
remote:        ├─┬ express-validator@3.2.0
remote:        │ ├── @types/bluebird@3.0.37
remote:        │ ├─┬ @types/express@4.0.35
remote:        │ │ ├─┬ @types/express-serve-static-core@4.0.44
remote:        │ │ │ └── @types/node@7.0.15
remote:        │ │ └─┬ @types/serve-static@1.7.31
remote:        │ │   └── @types/mime@0.0.29
remote:        │ ├── bluebird@3.5.0
remote:        │ ├── lodash@4.17.4
remote:        │ └── validator@6.2.1
remote:        ├─┬ jade@1.11.0
remote:        │ ├── character-parser@1.2.1
remote:        │ ├─┬ clean-css@3.4.25
remote:        │ │ ├─┬ commander@2.8.1
remote:        │ │ │ └── graceful-readlink@1.0.1
remote:        │ │ └─┬ source-map@0.4.4
remote:        │ │   └── amdefine@1.0.1
remote:        │ ├── commander@2.6.0
remote:        │ ├─┬ constantinople@3.0.2
remote:        │ │ └── acorn@2.7.0
remote:        │ ├─┬ jstransformer@0.0.2
remote:        │ │ ├── is-promise@2.1.0
remote:        │ │ └─┬ promise@6.1.0
remote:        │ │   └── asap@1.0.0
remote:        │ ├─┬ mkdirp@0.5.1
remote:        │ │ └── minimist@0.0.8
remote:        │ ├─┬ transformers@2.1.0
remote:        │ │ ├─┬ css@1.0.8
remote:        │ │ │ ├── css-parse@1.0.4
remote:        │ │ │ └── css-stringify@1.0.5
remote:        │ │ ├─┬ promise@2.0.0
remote:        │ │ │ └── is-promise@1.0.1
remote:        │ │ └─┬ uglify-js@2.2.5
remote:        │ │   ├─┬ optimist@0.3.7
remote:        │ │   │ └── wordwrap@0.0.3
remote:        │ │   └── source-map@0.1.43
remote:        │ ├─┬ uglify-js@2.8.22
remote:        │ │ ├── source-map@0.5.6
remote:        │ │ ├── uglify-to-browserify@1.0.2
remote:        │ │ └─┬ yargs@3.10.0
remote:        │ │   ├── camelcase@1.2.1
remote:        │ │   ├─┬ cliui@2.1.0
remote:        │ │   │ ├─┬ center-align@0.1.3
remote:        │ │   │ │ ├─┬ align-text@0.1.4
remote:        │ │   │ │ │ ├─┬ kind-of@3.2.0
remote:        │ │   │ │ │ │ └── is-buffer@1.1.5
remote:        │ │   │ │ │ ├── longest@1.0.1
remote:        │ │   │ │ │ └── repeat-string@1.6.1
remote:        │ │   │ │ └── lazy-cache@1.0.4
remote:        │ │   │ ├── right-align@0.1.3
remote:        │ │   │ └── wordwrap@0.0.2
remote:        │ │   ├── decamelize@1.2.0
remote:        │ │   └── window-size@0.1.0
remote:        │ ├── void-elements@2.0.1
remote:        │ └─┬ with@4.0.3
remote:        │   ├── acorn@1.2.2
remote:        │   └── acorn-globals@1.0.9
remote:        ├─┬ method-override@2.3.8
remote:        │ └── debug@2.6.3
remote:        ├─┬ mongoose@4.9.7
remote:        │ ├── async@2.1.4
remote:        │ ├── bson@1.0.4
remote:        │ ├── hooks-fixed@2.0.0
remote:        │ ├── kareem@1.4.1
remote:        │ ├─┬ mongodb@2.2.26
remote:        │ │ ├── es6-promise@3.2.1
remote:        │ │ ├─┬ mongodb-core@2.1.10
remote:        │ │ │ └─┬ require_optional@1.0.0
remote:        │ │ │   ├── resolve-from@2.0.0
remote:        │ │ │   └── semver@5.3.0
remote:        │ │ └─┬ readable-stream@2.2.7
remote:        │ │   ├── buffer-shims@1.0.0
remote:        │ │   ├── core-util-is@1.0.2
remote:        │ │   ├── isarray@1.0.0
remote:        │ │   ├── process-nextick-args@1.0.7
remote:        │ │   ├── string_decoder@1.0.0
remote:        │ │   └── util-deprecate@1.0.2
remote:        │ ├── mpath@0.2.1
remote:        │ ├── mpromise@0.5.5
remote:        │ ├─┬ mquery@2.3.0
remote:        │ │ ├── bluebird@2.10.2
remote:        │ │ ├─┬ debug@2.2.0
remote:        │ │ │ └── ms@0.7.1
remote:        │ │ └── sliced@0.0.5
remote:        │ ├── ms@0.7.2
remote:        │ ├── muri@1.2.1
remote:        │ ├── regexp-clone@0.0.1
remote:        │ └── sliced@1.0.1
remote:        ├─┬ morgan@1.8.1
remote:        │ ├── basic-auth@1.1.0
remote:        │ ├── debug@2.6.1
remote:        │ └── on-headers@1.0.1
remote:        └─┬ serve-favicon@2.4.2
remote:        └── ms@1.0.0
remote:
remote:
remote: -----> Caching build
remote:        Clearing previous node cache
remote:        Saving 2 cacheDirectories (default):
remote:        - node_modules
remote:        - bower_components (nothing to cache)
remote:
remote: -----> Build succeeded!
remote: -----> Discovering process types
remote:        Procfile declares types     -> (none)
remote:        Default types for buildpack -> web
remote:
remote: -----> Compressing...
remote:        Done: 16.7M
remote: -----> Launching...
remote:        Released v4
remote:        https://daylogapp-node.herokuapp.com/ deployed to Heroku
remote:
remote: Verifying deploy... done.
To https://git.heroku.com/daylogapp-node.git
 * [new branch]      master -> master
~~~

### Check if working

Visit the URL provided in the logs, visit app-name.herokuapp.com, or type `heroku open` in the console to make sure everything works as intended with the following page at the start:

![DayLogApp Node.js index page](/assets/images/posts/2017-04-24-day-log-app-nodejs-part-1/index-page.png){:class="img-responsive"}

This application is available at [`https://daylogapp-node.herokuapp.com/`](https://daylogapp-node.herokuapp.com/).

### Update files in Heroku `master` branch

After the initial `push` to the Heroku `master` branch there will be a need to update the files. Sometimes if we try the same command `git push heroku master` we will get the message:

    Everything up-to-date

This is not what we should expect - must be `git push` logs.

Use the following instead to force delivery of changes:

    git push heroku HEAD:master

## References
* Blum, Roman. "Web process failed to bind to $PORT within 60 seconds of launch · Issue #119 · Naltox/telegram-node-bot." *GitHub*. N.p., 20 Sept. 2016. Web. 01 May 2017. <[`https://github.com/Naltox/telegram-node-bot/issues/119#issuecomment-248320492`](https://github.com/Naltox/telegram-node-bot/issues/119#issuecomment-248320492)>.
* "Deploying Node.js Apps on Heroku." *Heroku Dev Center*. N.p., 24 Aug. 2016. Web. 30 Apr. 2017. <[`https://devcenter.heroku.com/articles/deploying-nodejs`](https://devcenter.heroku.com/articles/deploying-nodejs)>.
* fivepointseven. "Heroku Node.js Error R10 (Boot timeout) -> Web process failed to bind to $PORT within 60 seconds of launch." *Stack Overflow*. N.p., 27 June 2015. Web. 04 May 2017. <[`http://stackoverflow.com/a/31094668`](http://stackoverflow.com/a/31094668)>.
* Johnsen, Chris. "How to push different local Git branches to Heroku/master." *Stack Overflow*. N.p., 5 June 2010. Web. 02 May 2017. <[`http://stackoverflow.com/a/2980050`](http://stackoverflow.com/a/2980050)>.
* Lollar, Ian. "Heroku node.js error (Web process failed to bind to $PORT within 60 seconds of launch)." *Stack Overflow*. N.p., 28 Mar. 2013. Web. 01 May 2017. <[`http://stackoverflow.com/a/15693371`](http://stackoverflow.com/a/15693371)>.
* "Object Modeling in Node.js with Mongoose." *Heroku Dev Center*. N.p., 24 Mar. 2017. Web. 30 Apr. 2017. <[`https://devcenter.heroku.com/articles/nodejs-mongoose`](https://devcenter.heroku.com/articles/nodejs-mongoose)>.