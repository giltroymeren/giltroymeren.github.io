---
layout: post
title:  "How to change Express's default template engine from Jade to EJS"
date:   2017-04-22
categories: notes node.js express jade ejs
---

I am currently studying [node.js](https://nodejs.org/en/) development with [Express](https://expressjs.com/) as the framework. The default template engine is Jade (renamed to [Pug](https://pugjs.org/api/getting-started.html)) and wanted to use [EmbeddedJS](http://www.embeddedjs.com/) instead.

## Steps

1.  Make sure that [npm](https://www.npmjs.com/) in installed in your system.

        > npm -v
        3.10.3

    Please install if you have not.

2.  Install Express

        > npm install -g express
        /usr/local/lib
        └─┬ express@4.15.2
          └─┬ finalhandler@1.0.2
            └─┬ debug@2.6.4
              └── ms@0.7.3

    The `-g` indicates that it will be install globally so that the method `express` can be used.

3.  Create the project

    ~~~ console
    > express <app-name>

      warning: the default view engine will not be jade in future releases
      warning: use `--view=jade' or `--help' for additional options


       create : app-name
       create : app-name/package.json
       create : app-name/app.js
       create : app-name/public
       create : app-name/routes
       create : app-name/routes/index.js
       create : app-name/routes/users.js
       create : app-name/views
       create : app-name/views/index.jade
       create : app-name/views/layout.jade
       create : app-name/views/error.jade
       create : app-name/bin
       create : app-name/bin/www
       create : app-name/public/javascripts
       create : app-name/public/images
       create : app-name/public/stylesheets
       create : app-name/public/stylesheets/style.css

       install dependencies:
         $ cd app-name && npm install

       run the app:
         $ DEBUG=app-name:* npm start
    ~~~

    This will create the folder `app-name` with the files seen in the logs.

4.  Install dependencies

    Node projects have a `package.json` that contains the project's dependency information. Run `npm install` inside the project directory to install the dependencies. The folder `node_modules` should be available after execution.

    > **NOTE**: Add `node_modules` to your `.gitignore`.

5.  Check if the project works out-of-the-box

        > npm start
        app-name@0.0.0 start /path/to/app-name
        node ./bin/www

    Visit `http://localhost:3000/` in your browser (defined in `/bin/www`) and the page should be similar to the following:

    ![Express index page](/images/posts/2017-04-22-how-to-change-express-jade-to-ejs/express-index-page.png){:class="img-responsive"}

    Now we can start with adding EJS dependency!

6.  Install EJS

    a.  Add EJS as a dependency

        > npm install ejs --save

    This will add an entry EJS in `package.json` and add EJS files in `node_modules`.

    b.  Change app configuration

    Go to `app.js` and change the following line from

        app.set('view engine', 'jade');

    to

        app.set('view engine', 'ejs');

    c. Change file extensions

    The `views` folder contains the view files. Change their file extension from `.jade` to `.ejs`.

    If we run the server again (to reflect the changes made in `app.js`), then the Jade-style content of the renamed files should appear as normal HTML text.

    The [EJS documentation](http://ejs.co/) has the details on how to use the template engine.

