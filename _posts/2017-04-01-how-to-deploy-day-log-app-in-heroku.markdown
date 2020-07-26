---
layout: post
title:  "How to deploy \"Day Log App\" in Heroku"
date:   2017-04-01
categories: projects php laravel notes heroku deployment daylogapp
---

The [*Day Log App*]({{ site.baseurl }}{% post_url 2017-03-20-day-log-app %}) is currently hosted by [Heroku](http://heroku.com). This article will show the steps I performed in order to successfully deploy the application.

## Table of Contents
* TOC
{:toc}

## Deploy existing project to Heroku

The [steps](https://devcenter.heroku.com/articles/getting-started-with-laravel#deploying-to-heroku) to deploy to Heroku are still the same except for the initial Laravel project creation.

The following are the commands I used and their resulting logs during my actual run through as additional reference.

### Create Heroku application

#### Create `Procfile`

    > echo web: vendor/bin/heroku-php-apache2 public/ > Procfile
    > git add .
    > git commit -m "Profile for Heroku"

This `Procfile` should contain something similar to:

    web: vendor/bin/heroku-php-apache2 public/

#### Initialize application

    > heroku create
    Creating app... done, ⬢ your-app-name
    https://your-app-name.herokuapp.com/ | https://git.heroku.com/your-app-name.git

#### Set `APP_KEY` variable

    > heroku config:set APP_KEY=$(php artisan --no-ansi key:generate --show)
    Setting APP_KEY and restarting ⬢ your-app-name... done, v3
    APP_KEY: base64:<APP_KEY>

#### `push` source to Heroku

~~~ console
> git push heroku master
Counting objects: 854, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (783/783), done.
Writing objects: 100% (854/854), 819.50 KiB | 0 bytes/s, done.
Total 854 (delta 507), reused 0 (delta 0)
remote: Compressing source files... done.
remote: Building source:
remote:
remote:  !     Warning: Multiple default buildpacks reported the ability to handle this app. The first buildpack in the list below will be used.
remote:             Detected buildpacks: PHP,Node.js
remote:             See https://devcenter.heroku.com/articles/buildpacks#buildpack-detect-order
remote: -----> PHP app detected
remote: -----> Bootstrapping...
remote: -----> Installing platform packages...
remote:        - php (7.1.3)
remote:        - ext-mbstring (bundled with php)
remote:        - apache (2.4.20)
remote:        - nginx (1.8.1)
remote: -----> Installing dependencies...
remote:        Composer version 1.4.1 2017-03-10 09:29:45
remote:        Loading composer repositories with package information
remote:        Installing dependencies from lock file
remote:        Package operations: 34 installs, 0 updates, 0 removals
remote:          - Installing doctrine/inflector (v1.1.0): Downloading (100%)
remote:          - Installing erusev/parsedown (1.6.1): Downloading (100%)
remote:          - Installing jakub-onderka/php-console-color (0.1): Downloading (100%)
remote:          - Installing symfony/process (v3.2.6): Downloading (100%)
remote:          - Installing symfony/polyfill-mbstring (v1.3.0): Downloading (100%)
remote:          - Installing psr/log (1.0.2): Downloading (100%)
remote:          - Installing symfony/debug (v3.2.6): Downloading (100%)
remote:          - Installing symfony/console (v3.2.6): Downloading (100%)
remote:          - Installing symfony/translation (v3.2.6): Downloading (100%)
remote:          - Installing nesbot/carbon (1.22.1): Downloading (100%)
remote:          - Installing vlucas/phpdotenv (v2.4.0): Downloading (100%)
remote:          - Installing symfony/css-selector (v3.2.6): Downloading (100%
remote:          - Installing tijsverkoyen/css-to-inline-styles (2.2.0): Downloading (100%)
remote:          - Installing symfony/var-dumper (v3.2.6): Downloading (100%)
remote:          - Installing symfony/routing (v3.2.6): Downloading (100%)
remote:          - Installing symfony/http-foundation (v3.2.6): Downloading (100%)
remote:          - Installing symfony/event-dispatcher (v3.2.6): Downloading (100%)
remote:          - Installing symfony/http-kernel (v3.2.6): Downloading (100%)
remote:          - Installing symfony/finder (v3.2.6): Downloading (100%)
remote:          - Installing swiftmailer/swiftmailer (v5.4.6): Downloading (100%)
remote:          - Installing paragonie/random_compat (v2.0.9): Downloading (100%)
remote:          - Installing ramsey/uuid (3.5.2): Downloading (100%)
remote:          - Installing mtdowling/cron-expression (v1.2.0): Downloading (100%)
remote:          - Installing monolog/monolog (1.22.1): Downloading (100%)
remote:          - Installing league/flysystem (1.0.35): Downloading (100%)
remote:          - Installing laravel/framework (v5.4.15): Downloading (100%)
remote:          - Installing facebook/webdriver (1.3.0): Downloading (100%)
remote:          - Installing laravel/dusk (v1.0.10): Downloading (100%)
remote:          - Installing nikic/php-parser (v3.0.5): Downloading (100%)
remote:          - Installing jakub-onderka/php-console-highlighter (v0.3.2): Downloading (100%)
remote:          - Installing dnoegel/php-xdg-base-dir (0.1): Downloading (100%)
remote:          - Installing psy/psysh (v0.8.2): Downloading (100%)
remote:          - Installing laravel/tinker (v1.0.0): Downloading (100%)
remote:          - Installing laravelcollective/html (v5.4.1): Downloading (100%)
remote:        Generating optimized autoload files
remote:        > Illuminate\Foundation\ComposerScripts::postInstall
remote:        > php artisan optimize
remote:        Generating optimized class loader
remote:        The compiled services file has been removed.
remote: -----> Preparing runtime environment...
remote: -----> Checking for additional extensions to install...
remote: -----> Discovering process types
remote:        Procfile declares types -> web
remote:
remote: -----> Compressing...
remote:        Done: 27.5M
remote: -----> Launching...
remote:        Released v4
remote:        https://your-app-name.herokuapp.com/ deployed to Heroku
remote:
remote: Verifying deploy... done.
To https://git.heroku.com/your-app-name.git
 * [new branch]      master -> master
~~~

##### Manage branches

The current Git branch during the deployment above was performed in `master`. `master -> master` simply referes to the Git `master` branch pushed to the Heroku `master` branch.

However if you wish to use another branch then the output will be similar to:

    To https://git.heroku.com/your-app-name.git
     * [new branch]      your_custom_branch_name -> master

I used a different branch in this case because the local and production have different configurations (mainly in the database connection settings). This is a convenient way to separate Heroku-related changes from the rest of the source code.

In order to do this, `git push heroku master` should not be used; instead [use](http://stackoverflow.com/a/11143639):

    git push heroku <your_custom_branch_name>:master

> **NOTE**: The content of `https://git.heroku.com/your-app-name.git` is the same as the branch where the code was pushed

### Migrate database

Once the source code has been pushed, we can visit its URL provided in the logs. For example:

    remote: -----> Launching...
    remote:        Released v4
    remote:        https://your-app-name.herokuapp.com/ deployed to Heroku
    remote:
    remote: Verifying deploy... done.

If we visit `https://your-app-name.herokuapp.com` we will be greeted by the following page:

![Laravel Heroku 404 page](/images/posts/2017-04-01-how-to-deploy-day-log-app-in-heroku/laravel-heroku-404-page.png){:class="img-responsive"}

This is because we do not have the database set up yet. Heroku's free database add-on [ClearDB](https://devcenter.heroku.com/articles/cleardb) was used to perform the set up.

#### Add ClearDB to your application

    > heroku addons:create cleardb:ignite
    Creating cleardb:ignite on ⬢ your-app-name... free
    Created cleardb-angular-25571 as CLEARDB_DATABASE_URL
    Use heroku addons:docs cleardb to view documentation

#### Get database URL

    > heroku config | grep CLEARDB_DATABASE_URL
    CLEARDB_DATABASE_URL: mysql://<username>:<password>@<host_name>/<database_name>?reconnect=true

#### Set database URL

Just set the value of `CLEARDB_DATABASE_URL` in:

    heroku config:set DATABASE_URL='CLEARDB_DATABASE_URL'

This should result to:

    > heroku config:set DATABASE_URL='mysql://<username>:<password>@<host_name>/<database_name>?reconnect=true'
    Setting DATABASE_URL and restarting ⬢ your-app-name... done, v6
    DATABASE_URL: mysql://b5de37b05f01e6:a2bafc81@us-cdbr-iron-east-03.cleardb.net/heroku_1d6b4ea8b701a5e?reconnect=true

Now we have a registered Heroku database.

#### Migrate Laravel migrations

##### Set database configuration

We need to parse the variable `CLEARDB_DATABASE_URL` and set it to our database configuration file. In `config/database.php`, add the following before the `return` line:

~~~ ruby
$url = parse_url(getenv("CLEARDB_DATABASE_URL"));

$host = $url["host"];
$username = $url["user"];
$password = $url["pass"];
$database = substr($url["path"], 1);
~~~

##### Run command

Our migration is now ready. Just perform:

    > heroku run php /app/artisan migrate
    Running php /app/artisan migrate on ⬢ your-app-name.. up, run.9914 (Free)
    **************************************
    *     Application In Production!     *
    **************************************

     Do you really wish to run this command? (yes/no) [no]:
     > yes

    Migration table created successfully.
    Migrated: 2014_10_12_000000_create_users_table
    Migrated: 2014_10_12_100000_create_password_resets_table
    Migrated: YYYY_MM_DD_HHMMSS_create_daylogs_and_tasks_tables

Congratulations! Now you can use the Day Log App in Heroku.

### Miscellaneous

#### Rename application

Heroku provides randomly-generated names for its application. We can [change](https://devcenter.heroku.com/articles/renaming-apps#renaming-without-a-checkout) this by executing:

    > heroku apps:rename your-new-app-name
    Renaming your-old-app-name to your-new-app-name... done
    https://your-new-app-name.herokuapp.com/ | https://git.heroku.com/your-new-app-name.git
    Git remote heroku updated
     ▸    Don't forget to update git remotes for all other local checkouts of the app.

This will update both the local and deployed URLs of your application.

#### View content of database

There is no native online feature in Heroku to see the content of our database similar to [phpMyAdmin](https://www.phpmyadmin.net/) (as far as I know). Fortunately there are other ways to achieve this as explained in the ClearDB dashboard page.

Just go to the "Overview" tab of your project and click the "ClearDB MySQL" link under the "Installed add-ons" panel.

![Heroku "Installed add-ons" panel](/images/posts/2017-04-01-how-to-deploy-day-log-app-in-heroku/heroku-overview-cleardb.png){:class="img-responsive"}

On the page we will see the following message [[Alternative reference]](http://w2.cleardb.net/faqs/#general_17):

> **Managing Your Databases**

> If you want to manage the tables, data and other settings in your ClearDB database, we recommend that you use commonly known Graphical tools such as Oracle's MySQL Workbench, Sequel Pro for Mac OS X, or Navicat. You can also use any of the bundled MySQL tools, such as the mysql and mysqldump command-line utilities.

Just use one of those tools compatible to your system and refer to the structure of the generated ClearDB URL:

    mysql://<username>:<password>@<host_name>/<database_name>?reconnect=true

#### Resolve "max key length" error

This is an optional step.

During my first migration I did not encounter any problems but after that this particular one occurred multiple times and I cannot trace what could have caused it because I did not make any changes in related codes and deployment steps.

Whenever I execute the `migrate` command, I get this exact message:

    [Illuminate\Database\QueryException]
      SQLSTATE[42000]: Syntax error or access violation: 1071 Specified key was too long; max key
       length is 767 bytes (SQL: alter table `users` add unique `users_email_unique`(`email`))

The [working solution](https://laracasts.com/discuss/channels/laravel/laravel-54-failing-on-php-artisan-migrate-after-php-artisan-makeauth/replies/311688) in my case is to override the default maximum string length in `app\Providers\AppServerProvider.php`:

~~~ ruby
use Illuminate\Support\Facades\Schema;

public function boot()
{
    Schema::defaultStringLength(191);
}
~~~

This occurs on `< 5.7.7` MySQL databases because of a change in character set. The [documentation](https://github.com/laravel/docs/blob/5.4/migrations.md#index-lengths--mysql--mariadb) has an explanation for this issue.

## References

* "ClearDB MySQL." *Heroku Dev Center*. N.p., n.d. Web. 01 Apr. 2017. \<[`https://devcenter.heroku.com/articles/cleardb`](https://devcenter.heroku.com/articles/cleardb)\>.
* "Getting Started with Laravel on Heroku." *Heroku Dev Center*. N.p., n.d. Web. 31 Mar. 2017. \<[`https://devcenter.heroku.com/articles/getting-started-with-laravel#deploying-to-heroku`](https://devcenter.heroku.com/articles/getting-started-with-laravel#deploying-to-heroku)\>.
* "How do I manage my database(s) on ClearDB? Is there a phpMyAdmin or similar application that I can use?" *ClearDB*. N.p., n.d. Web. 01 Apr. 2017. \<[`http://w2.cleardb.net/faqs/#general_17`](http://w2.cleardb.net/faqs/#general_17)\>.
* Katzing, Nanichang. "Laravel 5.4 failing on php artisan migrate after php artisan make:auth." *Laracasts*. N.p., 26 Jan. 2017. Web. 01 Apr. 2017. \<[`https://laracasts.com/discuss/channels/laravel/laravel-54-failing-on-php-artisan-migrate-after-php-artisan-makeauth/replies/311688`](https://laracasts.com/discuss/channels/laravel/laravel-54-failing-on-php-artisan-migrate-after-php-artisan-makeauth/replies/311688)\>.
* Otwell, Taylor. "Laravel/docs: Index Lengths & MySQL / MariaDB." *GitHub*. N.p., 19 Jan. 2017. Web. 01 Apr. 2017. \<[`https://github.com/laravel/docs/blob/5.4/migrations.md#index-lengths--mysql--mariadb`](https://github.com/laravel/docs/blob/5.4/migrations.md#index-lengths--mysql--mariadb)\>.
* "Renaming Apps from the CLI." *Heroku Dev Center*. N.p., 15 Nov. 2016. Web. 01 Apr. 2017. \<[`https://devcenter.heroku.com/articles/renaming-apps#renaming-without-a-checkout`](https://devcenter.heroku.com/articles/renaming-apps#renaming-without-a-checkout)\>.
* Saldana, Javier. "How to push different local Git branches to Heroku/master." *Stack Overflow*. N.p., 21 June 2012. Web. 01 Apr. 2017. \<[`http://stackoverflow.com/a/11143639`](http://stackoverflow.com/a/11143639)>.
* Stauffer, Matt. "Laravel on Heroku - Using a MySQL database." *MattStauffer.co*. N.p., 1 May 2014. Web. 01 Apr. 2017. \<[`https://mattstauffer.co/blog/laravel-on-heroku-using-a-mysql-database`](https://mattstauffer.co/blog/laravel-on-heroku-using-a-mysql-database)\>.