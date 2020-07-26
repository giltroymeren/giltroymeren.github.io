---
layout: post
title:  "Day Log App"
date:   2017-03-20
updated: 2017-04-02
categories: projects php laravel notes daylogapp
---

This article is a combination of notes and guides taken from several [Laravel tutorials](#references) as I made the [*Day Log App*](http://daylogapp.herokuapp.com/) with the goal of learning Laravel development.

## Table of Contents
* TOC
{:toc}

## Prerequisites

Please refer to the official [README](https://bitbucket.org/giltroymeren/daylog/overview) for the dependencies and installation requirements.


## Goals

My PHP development before Laravel has been [CodeIgniter](https://codeigniter.com/) only and I have made several websites from it that were mostly forms-intensive. Additionally I have made a variety of web applications in Java using [Spring MVC](https://spring.io/) namely: a [prediction calculator](http://cas.upm.edu.ph:8080/xmlui/handle/123456789/41) and several internal tools in my current work. The design patterns I acquired from these two frameworks were my motivation in understanding Laravel.


### Concepts

The following were my target concepts to learn in Laravel as they represent the fundamental
requirements I usually get and personal workflow.

* Test-driven development (TDD) of front-end and back-end components using [PHPUnit](https://phpunit.de/) and [Laravel Dusk](https://laravel.com/docs/5.4/dusk)
* Database seeding
* [Route model binding](https://laravel.com/docs/5.4/routing#route-model-binding) (`@ModelAttribute` in Spring MVC)
* [Laravel Collective](https://laravelcollective.com/)
* Model-View-Controller pattern with Services and Data Access Objects (DAO) (Spring MVC)


## Development

The flow of the following "guide" is organized by TDD (red - failing test; green - create code to make the test pass; blue - refactor and repeat to red) and to follow the consolidated original flow in Flynsarmy and Barnes' guides.

Note that in Flynsarmy's both models `Projects` and `Tasks`'s CRUD from view to controller were done simultaneously while here they were done in succession because I wanted to handle each model's development separately.

In virtue of Flynsarmy's guide, instead of `Project`s with `Task`s, our project will be daily logs (think of an employee's daily work log but generalized) with children `Task`s.

### About the project

This project aims to create a daily log application that users can create, view, update or delete wherein each of these logs has their own tasks that can also be manipulated.

#### Models

We have three actors: `User`s, `Daylog`s and `Task`s and their relationships are:

* Each `User` can have many `Daylog`s
* Each `Daylog` can have zero to many `Task`s

> **NOTE**: The model `Daylog` was supposed to be `DayLog` but Laravel Eloquent parses `CamelCase` model names to `camel_case` table names when accessing the database. [[Reference]](http://stackoverflow.com/a/34781062)

The models have the following attributes:

* `Daylog`

      +------------+----------------------------------+------+-----+------------+----------------+
      | Field      | Type                             | Null | Key | Default    | Extra          |
      +------------+----------------------------------+------+-----+------------+----------------+
      | id         | int(10) unsigned                 | NO   | PRI | NULL       | auto_increment |
      | title      | varchar(1024)                    | NO   |     |            |                |
      | user_id    | int(10) unsigned                 | NO   | MUL | 0          |                |
      | slug       | varchar(255)                     | NO   |     |            |                |
      | location   | varchar(255)                     | NO   |     |            |                |
      | log_at     | date                             | NO   | UNI | 2017-03-15 |                |
      | category   | enum('ADEQUATE','MINOR','MAJOR') | NO   |     | NULL       |                |
      | created_at | timestamp                        | YES  |     | NULL       |                |
      | updated_at | timestamp                        | YES  |     | NULL       |                |
      +------------+----------------------------------+------+-----+------------+----------------+

* `Task`

      +------------+------------------+------+-----+----------+----------------+
      | Field      | Type             | Null | Key | Default  | Extra          |
      +------------+------------------+------+-----+----------+----------------+
      | id         | int(10) unsigned | NO   | PRI | NULL     | auto_increment |
      | daylog_id  | int(10) unsigned | NO   | MUL | 0        |                |
      | title      | varchar(255)     | NO   |     |          |                |
      | slug       | varchar(255)     | NO   |     |          |                |
      | start_at   | time             | NO   |     | 14:48:00 |                |
      | end_at     | time             | NO   |     | 14:48:00 |                |
      | completed  | tinyint(1)       | NO   |     | 0        |                |
      | created_at | timestamp        | YES  |     | NULL     |                |
      | updated_at | timestamp        | YES  |     | NULL     |                |
      +------------+------------------+------+-----+----------+----------------+

> **NOTE**: The fields `created_at` and `updated_at` are automatically generated through Laravel's
native authentication module which will be discussed later.

### Optional

#### Override default global style framework

By default Laravel's front-end framework is [Laravel Mix](https://laravel.com/docs/5.4/mix) and this can be overriden with a framework of your choice, say [Twitter Boostrap](http://getbootstrap.com/) (which is already available in `node_modules` folder).

Simply modify `resources/assets/sass/_variables.scss` by commenting or deleting all of its contents and re-run the SASS compiler.

### Guide proper

Go to your projects folder and execute:

    laravel new daylog

Make sure that the project is up and running initially by:

    php artisan serve

and you should see the URL (`127.0.0.1:8000` in this case) to visit on the generated logs depending on your sytem, for example:

    > php artisan serve
    Laravel development server started: <http://127.0.0.1:8000>
    [Tue Mar 14 19:08:24 2017] 127.0.0.1:51306 [200]: /favicon.ico

[comment]: # (Add picture of default Laravel 5.4 home page)

#### Add an authentication module

Laravel makes it easier for beginners to implement a user module. It has a native authentication module that can be easily added to the project by:

    php artisan make:auth

This should modify `routes/web.php` and create the following files:

[comment]: # (Explain what was modified in web.php)

* `app/Http/Controllers/HomeController.php`
* `resources/views/auth/`
* `resources/views/home.blade.php`
* `resources/views/layouts/`

Additionally a `users` table should be created in the database with the following structure:

[comment]: # (Confirm if the following table is actually created with the make:auth command)

    +----------------+------------------+------+-----+---------+----------------+
    | Field          | Type             | Null | Key | Default | Extra          |
    +----------------+------------------+------+-----+---------+----------------+
    | id             | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
    | name           | varchar(255)     | NO   |     | NULL    |                |
    | email          | varchar(255)     | NO   | UNI | NULL    |                |
    | password       | varchar(255)     | NO   |     | NULL    |                |
    | remember_token | varchar(100)     | YES  |     | NULL    |                |
    | created_at     | timestamp        | YES  |     | NULL    |                |
    | updated_at     | timestamp        | YES  |     | NULL    |                |
    +----------------+------------------+------+-----+---------+----------------+

When we visit the site again there should be new and working pages for "Login" and "Register" available.

#### Create migrations

First make sure that your database credentials are configured correctly in `.env` specifically the variables `DB_DATABASE`, `DB_USERNAME` and `DB_PASSWORD` and your MySQL (this case) is up.

We will prepare dummy data in our database in preparation for our first set of tests. Laravel has [database migrations](https://laravel.com/docs/5.4/migrations) that provides a convenient way to manage mass database manipulation. A [similar module](https://www.codeigniter.com/user_guide/libraries/migration.html) is also available in CodeIgniter however I have never used it before.

To create a migration for our project:

    php artisan make:migration create_daylogs_and_tasks_tables --create="daylogs"

The command will generate a file in `database/migrations` with the file name format "`YYYY_MM_DD_HHMMSS_create_daylogs_and_tasks_tables.php`".

Our planned structure of `Daylog` and `Task` is set to the migration by modifying the `up()` method:

~~~ ruby
Schema::create('daylogs', function (Blueprint $table) {
    $table->increments('id');
    $table->string('title', 1024)->default('');
    $table->integer('user_id')->unsigned()->default(0);
    $table->foreign('user_id')->references('id')->on('users');
    $table->string('slug')->default('');
    $table->string('location')->default('');
    $table->date('log_at')->unique()->default(date_format(new DateTime(), 'Y-m-d'));
    $table->enum('category', array('ADEQUATE', 'MINOR', 'MAJOR'));
    $table->timestamps();
});
~~~

~~~ ruby
Schema::create('tasks', function(Blueprint $table) {
    $table->increments('id');
    $table->integer('daylog_id')->unsigned()->default(0);
    $table->foreign('daylog_id')->references('id')->on('daylogs')->onDelete('cascade');
    $table->string('title')->default('');
    $table->string('slug')->default('');
    $table->time('start_at')->default(date_format(new DateTime(), 'H:i'));
    $table->time('end_at')->default(date_format(new DateTime(), 'H:i'));
    $table->boolean('completed')->default(false);
    $table->timestamps();
});
~~~

and do not forget to register their tables for `down()` for rollback:

~~~ ruby
Schema::dropIfExists('tasks');
Schema::dropIfExists('daylogs');
~~~

> **NOTE**: `$table->timestamps()` creates the date fields `created_at` and `updated_at`.

Next is to run our migration scripts:

    php artisan migrate

[comment]: # (Add success logs)

Afterwards check if the migration was created successfully in the database for the two tables:

    mysql> show tables;
    +-------------------+
    | Tables_in_daylogs |
    +-------------------+
    | daylogs           |
    | migrations        |
    | password_resets   |
    | tasks             |
    | users             |
    +-------------------+
    5 rows in set (0.00 sec)

    mysql> describe daylogs;
    +------------+----------------------------------+------+-----+------------+----------------+
    | Field      | Type                             | Null | Key | Default    | Extra          |
    +------------+----------------------------------+------+-----+------------+----------------+
    | id         | int(10) unsigned                 | NO   | PRI | NULL       | auto_increment |
    | title      | varchar(1024)                    | NO   |     |            |                |
    | user_id    | int(10) unsigned                 | NO   | MUL | 0          |                |
    | slug       | varchar(255)                     | NO   |     |            |                |
    | location   | varchar(255)                     | NO   |     |            |                |
    | log_at     | date                             | NO   | UNI | 2017-03-14 |                |
    | category   | enum('ADEQUATE','MINOR','MAJOR') | NO   |     | NULL       |                |
    | created_at | timestamp                        | YES  |     | NULL       |                |
    | updated_at | timestamp                        | YES  |     | NULL       |                |
    +------------+----------------------------------+------+-----+------------+----------------+
    9 rows in set (0.00 sec)

    mysql> describe tasks;
    +------------+------------------+------+-----+----------+----------------+
    | Field      | Type             | Null | Key | Default  | Extra          |
    +------------+------------------+------+-----+----------+----------------+
    | id         | int(10) unsigned | NO   | PRI | NULL     | auto_increment |
    | daylog_id  | int(10) unsigned | NO   | MUL | 0        |                |
    | title      | varchar(255)     | NO   |     |          |                |
    | slug       | varchar(255)     | NO   |     |          |                |
    | start_at   | time             | NO   |     | 15:37:00 |                |
    | end_at     | time             | NO   |     | 15:37:00 |                |
    | completed  | tinyint(1)       | NO   |     | 0        |                |
    | created_at | timestamp        | YES  |     | NULL     |                |
    | updated_at | timestamp        | YES  |     | NULL     |                |
    +------------+------------------+------+-----+----------+----------------+
    9 rows in set (0.00 sec)

Now we can start our development with TDD!

#### Prepare database seeders

We will begin by starting from the database. Seeders are helpful in simulating the entries of our models as they are passed from the front-end of our application to make sure it is working correctly as intended before proceeding to the views.

*   Create test case

    Create `tests/Unit/SeederTest.php` with the following test case:
    ~~~ ruby
    public function testDaylogsTable_shouldContainTitle()
    {
        $this->assertDatabaseHas('daylogs', ['title' => 'My Daylog Title']);
    }
    ~~~

    It is pretty descriptive: if we create a new `Daylog` with `title` "My Daylog Title" then there should be a row in the `daylogs` table with the said value.

    Run `phpunit` (some systems may use `./vendor/bin/phpunit`) and it should **FAIL** with the message:

        There was 1 failure:

        1) Tests\Unit\SeederTest::testDaylogsTable_shouldContainTitle
        Failed asserting that a row in the table [daylogs] matches the attributes {"title":"My Daylog Title"}.

        The table is empty.

    This is because we just created the `daylogs` and `tasks` table and they are currently empty.

    Seeders will provide the data to our database tables.

    Let's make this test case pass by creating the seeder for `daylog`:

        php artisan make:seeder DaylogsTableSeeder

    Add this seeder's class in `DatabaseSeeder.php` so it can be recognized:

    ~~~ ruby
    public function run()
    {
        $this->call(DaylogsTableSeeder::class);
    }
    ~~~

    In addition, a model is needed for this seeder:

        php artisan make:model DaylogModel

    `app/Daylog.php` should be available after and make sure to add its table name as an attribute:

        protected $table = 'daylogs';

*   Create fakers

    Since we need to seed our database, we need a convenient way to provide dummy data that corresponds to our attributes' types and limitations. Laravel has a library of ["fakers"](https://github.com/fzaninotto/Faker) which fulfills our basic requirements.

    Generally all model's fakers are set in `database/factories/ModelFactory.php` so we add the following method with fakers that correspond to the `Daylog`'s attributes:

    ~~~ ruby
    $factory->define(App\Daylog::class, function (Faker\Generator $faker) {
        return [
            'user_id' => $faker->numberBetween($min = 1, $max = 5),
            'title' => $faker->paragraph,
            'slug' => $faker->slug,
            'location' => $faker->streetAddress,
            'log_at' => $faker->unique()->dateTimeThisDecade($max = 'now',
                $timezone = date_default_timezone_get())->format('Y-m-d'),
            'category' => $faker->randomElement($array = array('ADEQUATE', 'MINOR', 'MAJOR'))
        ];
    });
    ~~~

    Some notes for this faker:

    > **NOTE**: `user_id` will have a number between 1 and 5 inclusive. This strictly depends on a `UsersTableSeeder` which generates five user accounts to be run before this seeder.

    ~~~ ruby
    public function run()
    {
        DB::table('users')->delete();

        for($i = 0; $i < 5; $i++)
        {
            factory(\App\User::class)->create();
        }
    }
    ~~~

    Here we initially empty the `users` table and provide five `User`s.

    > **NOTE**: `log_at`'s format corresponds to the MySQL `date` format of `YYYY-MM-DD`.

*   Seed the table

    Go back to `DaylogsTableSeeder` and use the faker we defined:

    ~~~ ruby
    public function run()
    {
        DB::table('daylogs')->delete();

        for($i = 0; $i < 10; $i++)
        {
            factory(\App\Daylog::class)->create();
        }
    }
    ~~~

    This `run()` method will always clear the `daylogs` table and create 10 faked `Daylog`s.

    To run the seeder just execute

        php artisan db:seed

    [comment]: # (Add success logs)

*   Make test pass

    We can create new `Daylog` fakers in tests by `factory(\App\Daylog::class)->create()` and this will be inserted in the database.

    In `SeederTest::testDaylogsTable_shouldContainTitle` we currently check if there exists a `title` in `daylogs` table with the value "My Daylog Title". The problem is that fakers are random data; we can override the fake `title` value by providing an array inside the `create()` method:

    ~~~ ruby
    public function testDaylogsTable_shouldContainTitle()
    {
        $title = "My Title";
        factory(\App\Daylog::class)->create([
            'title' => $title,
        ]);

        $this->assertDatabaseHas('daylogs', ['title' => $title]);
    }
    ~~~

    This should make the first database unit test **PASS**.

*   Test all `Daylog` attributes

    We can complete the first `Daylog`s table test to check all of its attributes if filled correctly.

    ~~~ ruby
    public function testDaylogsTable_shouldContainAllFields()
    {
        $userID = 1;
        $title = "My First Daylog";
        $slug = "my-first-daylog-title";
        $location = "ABC Street 123 Compound XYZ City";
        $log_at = "2017-03-15";
        $category = "ADEQUATE";

        $daylog = factory(\App\Daylog::class)->create([
            'user_id' => $userID,
            'title' => $title,
            'slug' => $slug,
            'location' => $location,
            'log_at' => $log_at,
            'category' => $category
        ]);

        $this->assertDatabaseHas('daylogs', [
            'user_id' => $userID,
            'title' => $title,
            'slug' => $slug,
            'location' => $location,
            'log_at' => $log_at,
            'category' => $category
        ]);

        \App\Daylog::destroy($daylog->id);
    }
    ~~~

    We have overriden all attributes with legal values and note that `destroy()` will delete our object after.

    Now that we have a fake we should take advantage of it instead to avoid declaring many variables:

    ~~~ ruby
    public function testDaylogsTable_shouldContainAllFields()
    {
        $daylog = factory(\App\Daylog::class)->create();

        $this->assertDatabaseHas('daylogs', [
            'user_id' => $daylog->user_id,
            'title' => $daylog->title,
            'slug' => $daylog->slug,
            'location' => $daylog->location,
            'log_at' => $daylog->log_at,
            'category' => $daylog->category
        ]);

        \App\Daylog::destroy($daylog->id);
    }
    ~~~

    [comment]: # (Check if asserting $daylog object is possible instead of enumerating all attributes)

*   Summary of creating a seeder

    Implementation of seeders by TDD are composed of the following files:

    * `tests/Unit/SeederTest.php`
    * `database/seeds/ModelTableSeeder.php`
    * `database/seeds/DatabaseSeeder.php`
    * `app/Model.php`
    * `database/factories/ModelFactory.php`

    where `Model` is the name of your model.

#### Browser test

In Flynsarmy's guide all `Project`s were shown in a starting page where users the CRUD options are available. This is also the structure in this application.

We have made our first test on the database. Now we can also make sure our user interface performs as intended with browser tests.

*   Install Laravel Dusk

    We will create *feature tests* using [Laravel Dusk](https://laravel.com/docs/5.4/dusk) for showing all available `Daylog`s in a page.

    Firstly we need to add [Composer](https://getcomposer.org/) dependency to Dusk (if it is not initially included in your project) in order to use browser-specific tests.

        composer require laravel/dusk

    Prepare Dusk dependencies for tests that will use it.

    * Register the `DuskServiceProvider` class in `app/Providers/AppServiceProvider.php`:

    ~~~ ruby
    public function register()
    {
        if ($this->app->environment('local', 'testing')) {
            $this->app->register(DuskServiceProvider::class);
        }
    }
    ~~~

    * Execute Dusk installation via Laravel:

        php artisan dusk:install

*   Create test case

    Create `tests/DuskTestCase.php` and add the following
    (as described in the [5.4 Dusk tutorial](https://laravel.com/docs/5.4/dusk#getting-started)):

    ~~~ ruby
    namespace Tests;

    use Laravel\Dusk\TestCase as BaseTestCase;
    use Facebook\WebDriver\Remote\RemoteWebDriver;
    use Facebook\WebDriver\Remote\DesiredCapabilities;

    abstract class DuskTestCase extends BaseTestCase
    {
        use CreatesApplication;

        /**
         * Prepare for Dusk test execution.
         *
         * @beforeClass
         * @return void
         */
        public static function prepare()
        {
            static::startChromeDriver();
        }

        /**
         * Create the RemoteWebDriver instance.
         *
         * @return \Facebook\WebDriver\Remote\RemoteWebDriver
         */
        protected function driver()
        {
            return RemoteWebDriver::create(
                'http://localhost:9515', DesiredCapabilities::chrome()
            );
        }
    }
    ~~~

    We can now use `DuskTestCase` to test our pages! Prepare `tests/Feature/DaylogTest.php` with content similar to the Seeder test that we have done and instead of accessing the database, we will check the view DOM elements.

    ~~~ ruby
    namespace Tests\Feature;

    use Tests\TestCase;
    use Tests\DuskTestCase;
    use Laravel\Dusk\Chrome;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Illuminate\Foundation\Testing\DatabaseTransactions;

    class DaylogTest extends DuskTestCase
    {
        /**
         *
         * @return void
         */
        public function testVisitViewDaylog_shouldShowCreatedDaylogTitle()
        {
            $title = "My First Daylog";

            $daylog = factory(\App\Daylog::class)->create([
                'title' => "My First Daylog",
            ]);

            $this->browse(function ($browser) {
                $browser->visit('/')
                    ->assertSee("My First Daylog");
            });

            $daylog->delete();
        }
    }
    ~~~

    Basically this test creates a `Daylog` instance and checks if its custom `title` "My First Daylog" is present in the page of the path `/`.

    > **NOTE**: `visit('/')` depends on the value of `APP_URL` defined in `.env`. Please check it if the value is the same with the URL provided in the logs after executing `php artisan serve`.

    If we run this test case, it should open a browser and visit a URL to the application's home page. It should **FAIL** because we do not have a setup for a page that will show the created `Daylog`'s `title`.

*   Setup index view

    In CodeIgniter, [routes](https://www.codeigniter.com/userguide3/general/routing.html) of processes are setup with controller names and hardcoded in `application/config/routes.php`; Spring MVC has the same principle using the [`@RequestMapping` annotation](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/RequestMapping.html). Laravel has this feature as well which is more similar to CI's way.

    Currently there is no view that corresponds to visiting `/` so we will prepare this.

    Go to `routes/web.php` and temporarily add the following code to fetch all `Daylog` entries from the database (denoted by `all()`) and send them to the view file called `welcome` (refers to a file `welcome.blade.php`).

    ~~~ ruby
    Route::get('/', function () {
        $daylogs = \App\Daylog::all();
        return view('welcome', compact('daylogs'));
    });
    ~~~

    [`compact()`](http://php.net/manual/en/function.compact.php) is a PHP function that creates a 2D array of keys and values.

    Add the following code block to print all `Daylog`s found in the database as an unordered list in `welcome.blade.php`.

    ~~~ html
    <ul>
        @foreach ($daylogs as $daylog)
            <li>{% raw %}{{ $daylog->title }}{% endraw %}</li>
        @endforeach
    </ul>
    ~~~

    The `testVisitViewDaylog_shouldShowCreatedDaylogTitle()` case should **PASS** after.

*   Compare using models

    We can modify the test to use the faker we made earlier instead of checking each `Daylog` attribute. Just move the creation and deletion of the model inside the `browse` method and replace the expected seen object to the entire object:

    ~~~ ruby
    public function testVisitViewDaylog_shouldShowRawDaylog()
    {
        $this->browse(function ($browser) {
            $daylog = factory(\App\Daylog::class)->create();

            $browser->visit('/')
                ->assertSee($daylog);

            $daylog->delete();
        });
    }
    ~~~

    > **NOTE**: Remember to modify `visit('/')` and/or `assertSee($daylog)` whenever changes in the routes or view page were made to ensure it reflects what is expected.

#### Create `DaylogController`

Controllers in Laravel can be create manually in the folder `app/Http/Controllers` or by executing

    php artisan make:controller DaylogController

*   Set up routes

    In CI we have freedom to set our own route patterns and this is also possible in Laravel but the latter has a default set of routes so we do not have to redefine everything (unless the business case requires specific URL names different from the default or other personal choice).

    We can do this by setting a model as a resource route in `web.php`:

        Route::resource('daylogs', 'DaylogController');

    This is the way for Laravel to recognize our model and allocate the default routes which can be checked by executing:

        > php artisan route:list daylogs
        +--------+-----------+-------------------------------+------------------+------------------------------------------------------------------------+--------------+
        | Domain | Method    | URI                           | Name             | Action                                                                 | Middleware   |
        +--------+-----------+-------------------------------+------------------+------------------------------------------------------------------------+--------------+
        ...
        |        | GET|HEAD  | daylogs                       | daylogs.index    | App\Http\Controllers\DaylogController@index                            | web          |
        |        | POST      | daylogs                       | daylogs.store    | App\Http\Controllers\DaylogController@store                            | web          |
        |        | GET|HEAD  | daylogs/create                | daylogs.create   | App\Http\Controllers\DaylogController@create                           | web          |
        |        | GET|HEAD  | daylogs/{daylog}              | daylogs.show     | App\Http\Controllers\DaylogController@show                             | web          |
        |        | DELETE    | daylogs/{daylog}              | daylogs.destroy  | App\Http\Controllers\DaylogController@destroy                          | web          |
        |        | PUT|PATCH | daylogs/{daylog}              | daylogs.update   | App\Http\Controllers\DaylogController@update                           | web          |
        |        | GET|HEAD  | daylogs/{daylog}/edit         | daylogs.edit     | App\Http\Controllers\DaylogController@edit                             | web          |
        ...
        +--------+-----------+-------------------------------+------------------+------------------------------------------------------------------------+--------------+

    Since we also added a `slug` attribute to our model, we will set it as the URL to identify each of our `Daylog`s (instead of the initial `id` from the database) in the same file:

    ~~~ ruby
    Route::bind('daylogs', function($value, $route) {
        return App\Daylog::whereSlug($value)->first();
    });
    ~~~

    Now that we have set the routes, we will revert the index path to the original:

    ~~~ ruby
    Route::get('/', function () {
        return view('welcome');
    });
    ~~~

    because we will move the code to fetch a single `Daylog` to its own controller method later.

    We should also remove the `Daylog` loop we set earlier in `welcome.blade.php` because we will use a new view file.

    If we run our tests, expect that `testVisitViewDaylog_shouldShowRawDaylog()` should **FAIL**. We will comment it for now.

    [comment]: # (Resolve if this case is replaced by the next test case)

#### Set up `DaylogController` methods

We will now create each route-controller binding for `Daylog` to create a CRUD form for this model as follows:

##### `DaylogController::index()`

Create a new test case called `testVisitDaylogsIndexPage_shouldShowDaylogPageTitle()` that will check if the text "Available Day Logs" (page title) and a list with ID attribute "`list-daylogs`" are present in the URL `/daylogs`:

~~~ ruby
public function testVisitDaylogsIndexPage_shouldShowListOfDaylogs()
{
    $this->browse(function ($browser) {
        $browser->visit("/daylogs/")
            ->assertSee("Available Day Logs")
            ->assertVisible("#list-daylogs");

        $browser->with('#list-daylogs', function ($table) {
            $table->assertVisible('li');
        });
    });
}
~~~

The second call to `browser` checks if the container with ID "`list-daylogs`" has children.

> **NOTE**: Make sure that there are `Daylog`s in the database.

We will create the `index` method in the controller that corresponds to the URL `/daylogs`:

~~~ ruby
public function index()
{
    $daylogs = Daylog::all();
    return view('daylogs.index', compact('daylogs'));
}
~~~

If you remember the previous method declared in `web.php`, this is the exact content and was moved to the controller because I want the two files to only have their specific code blocks based on function (i.e. routes in `web.php` and model manipulation in the controller).

The `view()` method here accepts a name of a view file and data to pass to this file.

Create `/resources/views/daylogs/index.blade.php` and this should inherit the formatting from the files generated before with `php artisan make:auth` (default Laravel user authentication).

~~~ html
@extends('layouts.app')

@section('content')
    <h2>Available Day Logs</h2>

    @if ( !$daylogs->count() )
        You have no daylogs
    @else
        <ul id="list-daylogs">
            @foreach( $daylogs as $daylog )
                <li>
                    <a href="{% raw %}{{ route('daylogs.show', $daylog->slug) }}{% endraw %}">
                        {% raw %}{{ $daylog->title }}{% endraw %}
                    </a>
                </li>
            @endforeach
        </ul>
    @endif
@endsection
~~~

If we visit `/daylogs`, we should see a page with text "Available Day Logs" and a list of `Daylog`s' titles in `/daylogs/index.blade.php`.

This should make the test **PASS**.

![Day Log - Home](/images/posts/2017-03-20-day-log-app/steps/list-daylog.png){:class="img-responsive"}

###### Route model binding

Before we proceed to the other CRUD processes, we need to set the [Route Model Binding](https://laravel.com/docs/5.4/routing#route-model-binding) in the controller in order to conveniently pass whole models from the view to controller instead of `ID`s.

This is simply done by addding the following line in `web.php`:

    Route::model('daylogs', 'Daylog');

It also requires to add this method in its model:

~~~ ruby
public function getRouteKeyName()
{
    return 'slug';
}
~~~

Now that is set, all routes with the pattern `/daylogs/{daylog}/*`'s corresponding controller method should accept a `Daylog` object as a parameter.

##### `DaylogController::show()`

Create a test that checks when a `Daylog` link is clicked, it should show a page containing its title.

~~~ ruby
public function testVisitDaylogsIndexPage_clickingOneLink_shouldShowShowPage()
{
    $this->browse(function ($browser) {
        $daylog = factory(\App\Daylog::class)->create();

        $browser->visit($this->url)
            ->assertSeeLink($daylog->title)
            ->clickLink($daylog->title)
            ->assertPathIs($this->url.$daylog->slug)
            ->assertSee($daylog->title);

        $daylog->delete();
    });
}
~~~

Controller:

~~~ ruby
use App\Daylog;

...

public function show(Daylog $daylog)
{
    return view('daylogs.show', compact('daylog'));
}
~~~

View file `/views/daylogs/show.blade.php`:

~~~ html
@extends('layouts.app')

@section('content')
    <h2>{% raw %}{{ $daylog->title }}{% endraw %}</h2>

    @if ( isset($daylog->tasks) && !$daylog->tasks->count() )
        @if ( $daylog->tasks->count() )
            <ul>
                @foreach( $daylog->tasks as $task )
                    <li>
                        <a href="{% raw %}{{ route('daylogs.tasks.show', [$daylog->slug, $task->slug]) }}{% endraw %}">
                            {% raw %}{{ $task->name }}{% endraw %}
                        </a>
                    </li>
                @endforeach
            </ul>
        @endif
    @else
        Your daylog has no tasks.
    @endif
@endsection
~~~

Since the `Task` model is not yet implemented we will only see the `Daylog`'s title in the page.

![Day Log - View with no Task](/images/posts/2017-03-20-day-log-app/steps/view-daylog-no-task.png){:class="img-responsive"}

##### `DaylogController::create()`

The next test will check for a create `Daylog` form page.

###### Set up `User` and `Daylog`'s' relationship

Remember that our rule is that each `User` can have many `Daylog`s and we will etablish this by adding the following:

* `User.php`:

~~~ ruby
public function daylogs()
{
    return $this->hasMany('App\Daylog');
}
~~~

* `Daylog.php`:

~~~ ruby
protected $guarded = [];

public function user()
{
    return $this->belongsTo('App\User');
}
~~~

* `DaylogController.php` to establish the authentication:

~~~ ruby
public function __construct()
{
    $this->middleware('auth');
}
~~~

> **NOTE**: This will cause the start of tests to be redirected to the login page. Make sure to add the necessary steps to pass such case. Here is an example of a login simulation (again, make sure the `User` that will be used in the tests exists in the database):

~~~ ruby
$this->browse(
    function ($browser)
    {
        $browser->visit('/login')
            ->type('email', <email>)
            ->type('password', <password>)
            ->press('Login');
    }
);
~~~

Depending on your application, adding this as a setup or in the first test case is enough (for the succeeding test cases to not redo login).

###### Laravel Collective

We will need to create forms for our create and edit features and fortunately Laravel Collective is available. It is an external library that provides templates for form fields and supports route-model binding.

You can add [`laravelcollective/html`](https://laravelcollective.com/docs/5.3/html) if your project does not have it yet by executing:

    composer require laravelcollective/html

and adding the following to `config/app.php` (in the `providers` and `aliases` arrays respectively):

    Collective\Html\HtmlServiceProvider::class

    ...

    'Form' => Collective\Html\FormFacade::class,
    'Html' => Collective\Html\HtmlFacade::class

Now we can use this library to set our forms.

###### Test form page

Make the test case which will just check if the form's labels are in the page:

~~~ ruby
public function testVisitDaylogsCreatePage_shouldShowCreateForm()
{
    $this->browse(function ($browser) {
        $createLinkText = 'Create Day Log';

        $browser->visit($this->url)
            ->assertSeeLink($createLinkText)
            ->clickLink($createLinkText)
            ->assertSee($createLinkText);

        $browser->with('form', function ($form) {
            $form->assertSee('Title')
                ->assertSee('Slug')
                ->assertSee('Location')
                ->assertSee('Date of Log')
                ->assertSee('Category');
        });
    });
}
~~~

Add the link to the create page in the `Daylog` index page:

    <p>
        {!! link_to_route('daylogs.create', 'Create Day Log') !!}
    </p>

The create page consists of two files:

* `resources/views/daylogs/create.blade.php`

~~~ html
@extends('layouts.app')

@section('content')
  <h2>Create Day Log</h2>

  {!! Form::model(new App\Daylog, ['route' => ['daylogs.store']]) !!}
      @include('daylogs/partials/_form', ['submit_text' => 'Create Day Log'])
  {!! Form::close() !!}
@endsection
~~~

* `resources/views/daylogs/partials/_form.blade.php`

~~~ html
<div class="form-group">
    {!! Form::label('title', 'Title') !!}
    {!! Form::textarea('title') !!}
</div>
<div class="form-group">
    {!! Form::label('slug', 'Slug') !!}
    {!! Form::text('slug') !!}
</div>
<div class="form-group">
    {!! Form::label('location', 'Location') !!}
    {!! Form::text('location') !!}
</div>
<div class="form-group">
    {!! Form::label('log_at', 'Date of Log') !!}
    {!! Form::date('log_at') !!}
</div>
<div class="form-group">
    {!! Form::label('category', 'Category') !!}
    {!! Form::select('category', [
      'ADEQUATE' => 'Adequate',
      'MINOR' => 'Minor',
      'MAJOR' => 'Major'
    ]) !!}
</div>
<div class="form-group">
    {!! Form::submit($submit_text, ['class'=>'btn primary']) !!}
</div>
~~~

`_form` corresponds to a reusable code block that can be shared with other view files. This will be useful for the other CRUD processes.

Controller:

~~~ ruby
public function create()
{
    return view('daylogs.create');
}
~~~

![Day Log - Create](/images/posts/2017-03-20-day-log-app/steps/create-daylog.png){:class="img-responsive"}

Now we have a form that will create a `Daylog` but currently it returns an error when we click the "Submit" button. Validation and storing to the database will be done by the `store()` method.

##### `DaylogController::store()`

This part will incorporate the native [validation](https://laravel.com/docs/5.4/validation) library to apply our own rules for the submitted `Daylog` in the create page.

We will modify the test `testVisitDaylogsCreatePage_shouldShowCreateForm()` because it does not fully test if the form and its fields are both existing and working. The test will require validation feedback from the controller that will serve as the requirement for passing the test case.

~~~ ruby
public function testCreateDaylogWithAllFieldsEmpty_shouldShowValidationErrorsForAllFields()
{
    $this->browse(
        function ($browser)
        {
            $createLinkText = 'Create Day Log';

            $browser->visit($this->url)
                ->assertSeeLink($createLinkText)
                ->clickLink($createLinkText)
                ->assertSee($createLinkText)
                ->press('Submit');
        }
    );
}
~~~

This test will fail because the method `store()` does not exist. Add this and our validation rules in the controller:

~~~ ruby
use Request;

...

protected $rules = [
    'title' => 'required|max:1024',
    'slug' => 'required|max:255',
    'location' => 'required|max:255',
    'log_at' => 'required|date_format:Y-m-d|unique:day_logs,log_at',
    'category' => 'required|in:ADEQUATE,MINOR,MAJOR',
];

...

public function store(\Illuminate\Http\Request $request)
{
    $this->validate($request, $this->rules);

    $input = Request::all();
    Daylog::create( $input );

    return redirect('daylogs')->with('message', 'Day Log created');
}
~~~

You can read more in the [Laravel Validation](https://laravel.com/docs/5.4/validation) chapter for more details.

Here are the cases for form submission:

* Failed:

    The following is just one of the many possibilities and combination of validation errors. Feel free to experiment using the other ones such as illegal format or exceeding character limits.

~~~ ruby
public function testCreateDaylogWithAllFieldsEmpty_shouldShowValidationErrorsForAllFields()
{
    $this->browse(
        function ($browser)
        {
            $createLinkText = 'Create Day Log';

            $browser->visit($this->url)
                ->assertSeeLink($createLinkText)
                ->clickLink($createLinkText)
                ->assertSee($createLinkText)
                ->press('Submit')
                ->assertSee('The title field is required.')
                ->assertSee('The slug field is required.')
                ->assertSee('The location field is required.')
                ->assertSee('The log at field is required.')
                ->assertSee('The category field is required.');
        }
    );
}
~~~

* Successful:

~~~ ruby
public function testCreateDaylogWithAllFieldsFilled_shouldShowOKMessageAndExistInHomePageAndDatabase()
{
    $this->browse(
        function ($browser)
        {
            $createLinkText = 'Create Day Log';
            $daylog = factory(\App\Daylog::class)->make();

            $browser->visit($this->indexURL)
                ->assertSeeLink($createLinkText)
                ->clickLink($createLinkText)
                ->assertSee($createLinkText)
                ->type('title', $daylog->title)
                ->type('slug', $daylog->slug)
                ->type('location', $daylog->location)
                ->value('#log_at', $daylog->log_at)
                ->select('category', $daylog->category)
                ->press('Submit')
                ->assertPathIs($this->indexURL)
                ->assertSeeLink($daylog->title);

            $this->assertDatabaseHas('daylogs', [
                'title' => $daylog->title,
                'slug' => $daylog->slug,
                'log_at' => $daylog->log_at,
            ]);
        }
    );
}
~~~

`make()` creates a static `Daylog` faker that will not be saved in the database unlike `create()` because we need the actual form to perform said operation.

##### `DaylogController:edit()`

Create a test to check if an edit form page shows up when a `Daylog` "EDIT" button in is clicked in the index page.

~~~ ruby
public function testEdit_shouldShowFormWithCorrectValues()
{
    $this->browse(
        function ($browser)
        {
            $daylog = factory(\App\Daylog::class)->create();

            $browser->visit($this->indexURL);

            $browser->with('[data-slug='.$daylog->slug.']',
                function($row) use ($daylog)
                {
                    $row->clickLink('EDIT')
                        ->assertPathIs($this->indexURL.'/'.$daylog->slug.'/edit');
                });

            $browser->with('form',
                function($form) use ($daylog)
                {
                    $form->assertInputValue('title', $daylog->title)
                        ->assertInputValue('slug', $daylog->slug)
                        ->assertInputValue('location', $daylog->location)
                        ->assertInputValue('log_at', $daylog->log_at)
                        ->assertSelected('category', $daylog->category);
                });

            $daylog->delete();
        }
    );
}
~~~

Controller:

~~~ ruby
public function edit(Daylog $daylog)
{
    return view('daylogs.edit', compact('daylog'));
}
~~~

View file `daylogs/edit.blade.php`:

~~~ html
@extends('layouts.app')

@section('content')
    <h2>Edit Day Log</h2>

    {!! Form::model($daylog, [
        'method' => 'PATCH',
        'route' => ['daylogs.update', $daylog->slug]]) !!}
        @include('daylogs/partials/_form')
    {!! Form::close() !!}
@endsection
~~~

![Day Log - Edit](/images/posts/2017-03-20-day-log-app/steps/edit-daylog.png){:class="img-responsive"}

##### `DaylogController::update`

We will modify the previous test to set the contents of the fields in the edit form.

~~~ ruby
public function testEditAndChangeTitleAndLogAt_shouldSeeNewTitleAndLogAtInHomePageAndDatabase()
{
    $this->browse(
        function ($browser)
        {
            $daylogCreate = factory(\App\Daylog::class)->create();
            $daylogMake = factory(\App\Daylog::class)->make();

            $browser->visit($this->indexURL);

            $browser->with('[data-slug='.$daylogCreate->slug.']',
                function($row) use ($daylogCreate)
                {
                    $row->clickLink('EDIT')
                        ->assertPathIs($this->indexURL.'/'.$daylogCreate->slug.'/edit');
                });

            $browser->with('form',
                function($form) use ($daylogCreate, $daylogMake)
                {
                    $form->assertInputValue('title', $daylogCreate->title)
                        ->assertInputValue('slug', $daylogCreate->slug)
                        ->assertInputValue('location', $daylogCreate->location)
                        ->assertInputValue('log_at', $daylogCreate->log_at)
                        ->assertSelected('category', $daylogCreate->category)

                        ->type('title', $daylogMake->title)
                        ->type('log_at', $daylogMake->log_at)
                        ->press('Submit');
                });

            $browser->assertSee($daylogMake->title);

            $this->assertDatabaseHas('daylogs', [
                'title' => $daylogMake->title,
                'log_at' => $daylogMake->log_at,
            ]);

            $daylogCreate->delete();
        }
    );
}
~~~

Controller method:

~~~ ruby
public function update(Daylog $daylog, \Illuminate\Http\Request $request)
{
    $this->validate($request, $this->rules);

    $input = array_except(Request::all(), '_method');
    $daylog->update($input);

    return redirect('daylogs/'.$daylog->slug)->with('message', 'Day Log updated.');
}
~~~

##### `DaylogController::destroy()`

Test:

~~~ ruby
public function testDelete_shouldNotBeInIndexPageAndDatabase()
{
    $this->browse(
        function ($browser)
        {
            $daylog = factory(\App\Daylog::class)->create();

            $browser->visit($this->indexURL);

            $browser->with('[data-slug='.$daylog->slug.']',
                function($row) use ($daylog)
                {
                    $row->press('DELETE');
                });

            $browser->assertDontSee($daylog->title);

            $this->assertDatabaseMissing('daylogs', [
                'id' => $daylog->id
            ]);
        }
    );
}
~~~

Controller:

~~~ ruby
public function destroy(Daylog $daylog)
{
    $daylog->delete();

    return redirect('daylogs')->with('message', 'Day Log deleted.');
}
~~~

Do not forget to add following button in the index page:

    ...

    {!! link_to_route('daylogs.edit', 'EDIT', array($daylog->slug), array('class' => 'btn btn-info')) !!},
    {!! Form::submit('DELETE', array('class' => 'btn btn-danger')) !!}

    ...

#### Implement `Task`s in `Daylog`s

As discussed before each `Daylog` can have zero or more `Task`s and we will gradually add this model to all the components in the application that need it.

*   Create database seeder

    * Create test

    In the `Daylog` seeder test before we checked if it was created successfully in the database. Currently it has no `Task` created alongside it so we will add another test that has this case. We will also modify the first with an assertion that no `Task` was created with it.

    ~~~ ruby
    public function testCreateDaylogWitTask_shouldBothBeInDatabase()
    {
        $daylog = factory(\App\Daylog::class)->create();
        $task = factory(\App\Task::class)->create([
                'daylog_id' => $daylog->id,
            ]);

        $this->assertDatabaseHas('daylogs', [
            'user_id' => $daylog->user_id,
            'title' => $daylog->title,
            'slug' => $daylog->slug,
            'location' => $daylog->location,
            'log_at' => $daylog->log_at,
            'category' => $daylog->category
        ]);

        $this->assertDatabaseHas('tasks', [
            'daylog_id' => $task->daylog_id,
            'title' => $task->title,
            'slug' => $task->slug,
            'start_at' => $task->start_at,
            'end_at' => $task->end_at,
            'completed' => $task->completed
        ]);

        $daylog->delete();
    }
    ~~~

    ~~~ ruby
    public function testCreateDaylogWithNoTask_shouldBeInDatabase()
    {
        $daylog = factory(\App\Daylog::class)->create();

        $this->assertDatabaseHas('daylogs', [
            'user_id' => $daylog->user_id,
            'title' => $daylog->title,
            'slug' => $daylog->slug,
            'location' => $daylog->location,
            'log_at' => $daylog->log_at,
            'category' => $daylog->category
        ]);

        $this->assertDatabaseMissing('tasks', [
            'daylog_id' => $daylog->id,
        ]);

        $daylog->delete();
    }
    ~~~

*   Seeder

    In `ModelFactory.php`

    ~~~ ruby
    $factory->define(App\Task::class, function (Faker\Generator $faker) {
        $start_at = $faker->time($format = 'H:i:s', $max = 'now');

        return [
            'daylog_id' => $faker->numberBetween($min = 1, $max = 5),
            'title' => $faker->paragraph,
            'slug' => $faker->slug,
            'start_at' => $start_at,
            'end_at' => $start_at + $faker->numberBetween($min = 1, $max = 12),
            'completed' => $faker->randomElement($array = array(true, false))
        ];
    });
    ~~~

    Generate seeder:

        php artisan make:seeder TasksTableSeeder

    Set the `Faker` in `run()`:

    ~~~ ruby
    public function run()
    {
        DB::table('tasks')->delete();

        for($i = 0; $i < 20; $i++)
        {
            factory(\App\Task::class)->create();
        }
    }
    ~~~

    Now both `Daylog` and `Task` will have dummy contents if we run:

        php artisan db:seed

*   Set up `Task` dependency to `Daylog`

    Create the model:

        php artisan make:model Task

    Add the following attributes and methods:

    ~~~ ruby
    protected $table = 'tasks';

    protected $guarded = [];

    public function getRouteKeyName()
    {
        return 'slug';
    }

    public function daylog()
    {
        return $this->belongsTo('App\Daylog');
    }
    ~~~

    Update `Daylog.php` with its relationship to `Task` objects:

    ~~~ ruby
    public function tasks()
    {
        return $this->hasMany('App\Task');
    }
    ~~~

*   Create controller

        php artisan make:controller TaskController

    Add the authentication protection in the constructor:

    ~~~ ruby
    public function __construct()
    {
        $this->middleware('auth');
    }
    ~~~

###### Bind `Task` to `Daylog`

Go to `web.php` and we will add the CRUD processing of `Task`s alongside each `Daylog`:

    Route::model('tasks', 'Task');

    ...

    Route::resource('daylogs.tasks', 'TasksController');

Wherein the updated routes should now be:

    > php artisan route:list

    +--------+-----------+------------------------------------+-----------------------+------------------------------------------------------------------------+--------------+
    | Domain | Method    | URI                                | Name                  | Action                                                                 | Middleware   |
    +--------+-----------+------------------------------------+-----------------------+------------------------------------------------------------------------+--------------+
    ...
    |        | POST      | daylogs/{daylog}/tasks             | daylogs.tasks.store   | App\Http\Controllers\TaskController@store                              | web          |
    |        | GET|HEAD  | daylogs/{daylog}/tasks             | daylogs.tasks.index   | App\Http\Controllers\TaskController@index                              | web          |
    |        | GET|HEAD  | daylogs/{daylog}/tasks/create      | daylogs.tasks.create  | App\Http\Controllers\TaskController@create                             | web          |
    |        | DELETE    | daylogs/{daylog}/tasks/{task}      | daylogs.tasks.destroy | App\Http\Controllers\TaskController@destroy                            | web          |
    |        | GET|HEAD  | daylogs/{daylog}/tasks/{task}      | daylogs.tasks.show    | App\Http\Controllers\TaskController@show                               | web          |
    |        | PUT|PATCH | daylogs/{daylog}/tasks/{task}      | daylogs.tasks.update  | App\Http\Controllers\TaskController@update                             | web          |
    |        | GET|HEAD  | daylogs/{daylog}/tasks/{task}/edit | daylogs.tasks.edit    | App\Http\Controllers\TaskController@edit                               | web          |
    ...
    +--------+-----------+------------------------------------+-----------------------+------------------------------------------------------------------------+--------------+

We can observe that it also has the similar set of controller methods as before, just that the model is within the parent's URL.

*   Set `Task` slug-based URL

    Add the following in `web.php`:

    ~~~ ruby
    Route::bind('tasks', function($value, $route) {
        return App\Task::whereSlug($value)->first();
    });
    ~~~

    > **NOTE**: We do not have to define a `getRouteKeyName()` method in the model because it was already added before.

*   Prepare browser test file

    Prepare the Dusk test in `tests/Feature/TaskTest.php`.

    ~~~ ruby
    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use Tests\DuskTestCase;
    use Laravel\Dusk\Chrome;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Illuminate\Foundation\Testing\DatabaseTransactions;

    class TaskTest extends DuskTestCase
    {
        //
    }
    ~~~

*   Show `Task`s in a `Daylog`'s view page

    Set up the first test case: when a `Daylog` with a `Task` is selected to be viewed, we should epect that it has `Task`s.

    ~~~ ruby
    private $indexURL = "/daylogs";
    private $loginURL = "/login";

    private $createLinkText = 'Create Day Log';

    public function testViewDaylogWithTask_shouldShowPageWithTasks()
    {
        $this->browse(
            function ($browser)
            {
                $daylog = factory(\App\Daylog::class)->create();
                $task = factory(\App\Task::class)->create([
                    'daylog_id' => $daylog->id,
                ]);

                $browser->visit($this->loginURL)
                    ->type('email', 'one@one.com')
                    ->type('password', '111111')
                    ->press('Login')

                    ->visit($this->indexURL)
                    ->assertSee("Available Day Logs")
                    ->assertVisible("#list-daylogs")
                    ->clickLink($daylog->title);

                $browser->with('#list-tasks',
                    function($list) use ($task)
                    {
                        $list->assertSee("Available Tasks")
                            ->assertSee($task->title);
                    }
                );

                $daylog->delete();
            }
        );
    }
    ~~~

    We will also modify a few tests in `DaylogTest` now that we have `Task`s in their view pages:

    ~~~ ruby
    public function testVisitIndexPage_clickingOneLink_shouldShowShowPage()

    ...

    $browser->visit($this->indexURL)
        ->assertDontSee("Available Tasks");

    ...

    ~~~

    Add the showing of list of `Task`s in `/daylogs/show.blade.php` (with modifications to the conditionals, because now we are sure all `Daylog`s have `Task`s:

    ~~~ html
    @if ( $daylog->tasks->count() )
        <h3>Available Tasks</h3>
        <ul id="list-tasks">
            @foreach( $daylog->tasks as $task )
                <li>
                    <a href="{% raw %}{{ route('daylogs.tasks.show', [$daylog->slug, $task->slug]) }}{% endraw %}">
                        {% raw %}{{ $task->title }}{% endraw %}
                    </a>
                </li>
            @endforeach
        </ul>
    @else
        Your daylog has no tasks.
    @endif
    ~~~

    ![Day Log - View with Tasks](/images/posts/2017-03-20-day-log-app/steps/view-daylog-yes-task.png){:class="img-responsive"}

#### Set up `TaskController` methods

##### `TaskController::index()`

In `DaylogController::index()`, it directs to a page with a list of all `Task`s which means we do not have to do separate view for this unless there is a need (or you want to).

We will just implement its method:

~~~ ruby
public function index(Task $task)
{
    return view('tasks.index', compact('task'));
}
~~~

##### `TaskController::show()`

Our test will check if the `Task` view page contains the title. Later we will further customize it to show its other attributes.

~~~ ruby
public function testViewTask_shouldShowPageWithTitle()
{
    $this->browse(
        function ($browser)
        {
            $daylog = factory(\App\Daylog::class)->create();
            $task = factory(\App\Task::class)->create([
                'daylog_id' => $daylog->id,
            ]);

            $browser->visit($this->indexURL)
                ->assertSee("Available Day Logs")
                ->assertVisible("#list-daylogs")
                ->clickLink($daylog->title)
                ->clickLink($task->title)
                ->assertSee($task->title);

            $daylog->delete();
        }
    );
}
~~~

Controller method:

~~~ ruby
use App\Daylog;
use App\Task;

...

public function show(Daylog $daylog, Task $task)
{
    return view('tasks.show', compact('daylog', 'task'));
}
~~~

View file:

~~~ html
@extends('layouts.app')

@section('content')
    <h2>
        {% raw %}{!! link_to_route('daylogs.show', 'Day Log: '.$daylog->title, [$daylog->slug]) !!}{% endraw %}
    </h2>
    <h3>{% raw %}{{ $task->title }}{% endraw %}</h3>
@endsection
~~~

![Task - View](/images/posts/2017-03-20-day-log-app/steps/view-task.png){:class="img-responsive"}

##### `TaskController::create()`

Create the test for the form page, wherein the "Create" link will be seen in every `Daylog` show page. We will only look for the form labels for now.

~~~ ruby
public function testCreate_shouldShowForm()
{
    $this->browse(
        function ($browser)
        {
            $daylog = factory(\App\Daylog::class)->create();

            $browser->visit($this->indexURL)
                ->assertSee("Available Day Logs")
                ->assertVisible("#list-daylogs")
                ->clickLink($daylog->title)
                ->clickLink("Create Task")
                ->assertSee("Title")
                ->assertSee("Slug")
                ->assertSee("Start Time")
                ->assertSee("End Time")
                ->assertSee("Completed");

            $daylog->delete();
        }
    );
}
~~~

Controller:

~~~ ruby
public function create(Daylog $daylog)
{
    return view('tasks.create', compact('daylog'));
}
~~~

View file `resources/views/tasks/create.blade.php`:

~~~ html
@extends('layouts.app')

@section('content')
    <h2>Create Task for Day Log "{% raw %}{{ $daylog->name }}{% endraw %}"</h2>

    {!! Form::model(new App\Task,
        ['route' => ['daylogs.tasks.store', $daylog->slug], 'class'=>'']) !!}
        @include('tasks/partials/_form', ['submit_text' => 'Submit'])
    {!! Form::close() !!}
@endsection
~~~

Together with `resources/views/partials/_form.blade.php`:

~~~ html
<div class="form-group">
    {!! Form::label('title', 'Title:') !!}
    {!! Form::text('title') !!}
</div>

<div class="form-group">
    {!! Form::label('slug', 'Slug:') !!}
    {!! Form::text('slug') !!}
</div>

<div class="form-group">
    {!! Form::label('start_at', 'Start Time:') !!}
    {!! Form::text('start_at') !!}
</div>

<div class="form-group">
    {!! Form::label('end_at', 'End Time:') !!}
    {!! Form::text('end_at') !!}
</div>

<div class="form-group">
    {!! Form::label('completed', 'Completed:') !!}
    {!! Form::checkbox('completed') !!}
</div>

<div class="form-group">
    {!! Form::submit($submit_text) !!}
</div>
~~~

![Task - Create](/images/posts/2017-03-20-day-log-app/steps/create-task.png){:class="img-responsive"}

##### `TaskController::store()`

Now that the form is ready we can modify the previous test case to actually create a `Task`. Same as before, we will create two: with and without validation errors.

~~~ ruby
public function testCreateWithAllEmpty_shouldShowValidationErrorsForAll()
{
    $this->browse(
        function ($browser)
        {
            $daylog = factory(\App\Daylog::class)->create();

            $browser->visit($this->indexURL)
                ->assertSee("Available Day Logs")
                ->assertVisible("#list-daylogs")
                ->clickLink($daylog->title)
                ->clickLink("Create Task")
                ->press("Submit")
                ->assertSee("The title is required")
                ->assertSee("The slug is required")
                ->assertSee("The start at is required")
                ->assertSee("The end at is required");

            $daylog->delete();
        }
    );
}
~~~

~~~ ruby
public function testCreateWithAllFilled_shouldShowOKMessageAndExistInHomePageAndDatabase()
{
    $this->browse(
        function ($browser)
        {
            $daylog = factory(\App\Daylog::class)->create();
            $task = factory(\App\Task::class)->make([
                'daylog_id' => $daylog->id,
            ]);

            $browser->visit($this->indexURL)
                ->assertSee("Available Day Logs")
                ->assertVisible("#list-daylogs")
                ->clickLink($daylog->title)
                ->clickLink($this->createLinkText)
                ->type('title', $task->title)
                ->type('slug', $task->slug)
                ->type('start_at', $task->start_at)
                ->type('end_at', $task->end_at)
                ->check('completed')
                ->press('Submit')
                ->assertPathIs($this->indexURL.'/'.$daylog->slug);

            $browser->with('#list-tasks',
                function($list) use ($task)
                {
                    $list->assertSeeLink($task->title);
                }
            );

            $this->assertDatabaseHas('tasks', [
                'title' => $task->title,
                'slug' => $task->slug,
            ]);
        }
    );
}
~~~

In order to make this always pass, we will make the `Task` faker for start and end time have hardcoded values:

    'start_at' => "10:10:10",
    'end_at' => "11:11:11",

[comment]: # (Resolve and confirm in actual code)

Create the method with validation to make this pass:

~~~ ruby
use Request; // replace the other one

...

protected $rules = [
    'title' => 'required|max:255',
    'slug' => 'required',
    'start_at' => 'required|date_format:H:i|before:end_at',
    'end_at' => 'required|date_format:H:i|before:start_at',
];

...

public function store(Daylog $daylog, \Illuminate\Http\Request $request)
{
    $this->validate($request, $this->rules);

    $input = Request::all();
    $input['daylog_id'] = $daylog->id;
    Task::create( $input );

    return redirect('daylogs/'.$daylog->slug)->with('message', 'Task created.');
}
~~~

##### `TaskController::edit()`

Test to see the edit page form with all fields's values matching the just-created `Daylog`:

~~~ ruby
public function testEdit_shouldSeeOldValuesInForm()
{
    $this->browse(
        function ($browser)
        {
            $daylog = factory(\App\Daylog::class)->create();
            $taskCreate = factory(\App\Task::class)->create([
                'daylog_id' => $daylog->id,
            ]);
            $taskMake = factory(\App\Task::class)->make([
                'daylog_id' => $daylog->id,
            ]);

            $browser->visit($this->indexURL)
                ->clickLink($daylog->title);

            $browser->with('[data-slug='.$taskCreate->slug.']',
                function($row) use ($daylog, $taskCreate)
                {
                    $row->clickLink('EDIT')
                        ->assertPathIs(
                            $this->indexURL.'/'.$daylog->slug
                            .'/tasks/'.$taskCreate->slug.'/edit');
                }
            );

            $browser->with('form',
                function($form) use ($taskCreate, $taskMake)
                {
                    $form->assertInputValue('title', $taskCreate->title)
                        ->assertInputValue('slug', $taskCreate->slug)
                        ->assertInputValue('start_at', $taskCreate->start_at)
                        ->assertInputValue('end_at', $taskCreate->end_at);

                    if($taskCreate->completed)
                    {
                        $form->assertChecked('completed');
                    }
                    else
                    {
                        $form->assertNotChecked('completed');
                    }
                }
            );

            $daylog->delete();
        }
    );
}
~~~

Controller:

~~~ ruby
public function edit(Daylog $daylog, Task $task)
{
    return view('tasks.edit', compact('daylog', 'task'));
}
~~~

View file `resources/views/tasks/edit.blade.php`:

~~~ html
@extends('layouts.app')

@section('content')
    <h2>Edit Task "{% raw %}{{ $task->name }}{% endraw %}"</h2>

    {!! Form::model($task, [
        'method' => 'PATCH',
        'route' => ['daylogs.tasks.update',
        $daylog->slug, $task->slug]]) !!}
        @include('tasks/partials/_form')
    {!! Form::close() !!}
@endsection
~~~

![Task - Edit](/images/posts/2017-03-20-day-log-app/steps/edit-task.png){:class="img-responsive"}

##### `TaskController::update()`

Create test for successful editing (validations were already covered by the create case):

~~~ ruby
public function testEditAndChangeTitle_shouldSeeNewTitleInHomePageAndDatabase()
{
    $this->browse(
        function ($browser)
        {
            $daylog = factory(\App\Daylog::class)->create();
            $taskCreate = factory(\App\Task::class)->create([
                'daylog_id' => $daylog->id,
            ]);
            $taskMake = factory(\App\Task::class)->make([
                'daylog_id' => $daylog->id,
            ]);

            $browser->visit($this->indexURL)
                ->clickLink($daylog->title);

            $browser->with('[data-slug='.$taskCreate->slug.']',
                function($row) use ($daylog, $taskCreate)
                {
                    $row->clickLink('EDIT')
                        ->assertPathIs(
                            $this->indexURL.'/'.$daylog->slug
                            .'/tasks/'.$taskCreate->slug.'/edit');
                }
            );

            $browser->with('form',
                function($form) use ($taskCreate, $taskMake)
                {
                    $form->type('title', $taskMake->title)
                        ->press('Submit');
                }
            );

            $browser->assertPathIs($this->indexURL.'/'.$daylog->slug
                    .'/tasks/'.$taskCreate->slug)
                ->assertSee($taskMake->title);


            $this->assertDatabaseHas('tasks', [
                'title' => $taskMake->title,
            ]);

            $daylog->delete();
        }
    );
}
~~~

Controller:

~~~ ruby
public function update(Daylog $daylog, Task $task, \Illuminate\Http\Request $request)
{
    $this->validate($request, $this->rules);

    $input = array_except(Request::all(), '_method');
    $task->update($input);

    return redirect('daylogs/'.$daylog->slug.'/tasks/'.$task->slug)
        ->with('message', 'Task updated.');
}
~~~

##### `TaskController::destroy()`

Test case:

~~~ ruby
public function testDelete_shouldNotBeInDaylogPageAndDatabase()
{
    $this->browse(
        function ($browser)
        {
            $daylog = factory(\App\Daylog::class)->create();
            $task = factory(\App\Task::class)->create([
                'daylog_id' => $daylog->id,
            ]);=

            $browser->visit($this->indexURL)
                ->clickLink($daylog->title);

            $browser->with('[data-slug='.$task->slug.']',
                function($row) use ($daylog, $task)
                {
                    $row->clickLink('DELETE')
                        ->assertPathIs(
                            $this->indexURL.'/'.$daylog->slug);
                }
            );

            $browser->assertDontSee($task->title);

            $this->assertDatabaseMissing('task', [
                'id' => $task->id
            ]);

            $daylog->delete();
        }
    );
}
~~~

Controller:

~~~ ruby
public function destroy(Daylog $daylog, Task $task)
{
    $task->delete();

    return redirect('daylogs/'.$daylog->slug)->with('message', 'Task deleted.');
}
~~~

Add the delete button in `resources/views/daylogs/show.blade.php`:

~~~ html

...

{!! link_to_route('daylogs.tasks.edit', 'EDIT',
        array($daylog->slug, $task->slug),
        array('class' => 'btn btn-info')) !!},
{!! Form::submit('DELETE', array('class' => 'btn btn-danger')) !!}

...

~~~

#### Miscellaneous

##### Create a helper class for notification messages

Currently we show notification messages after creating, updating or deleting our models by:

~~~ ruby
return redirect('daylogs')->with('message', 'Day Log created');
~~~

and it is duplicated in the other methods.

We can [create a helper class](http://stackoverflow.com/a/32772686) for `DaylogController` and `TaskController` that will contain the method to format this code. In the future, other helper methods can be put into this class as well.

First, create `ControllerHelper.php` in `app/Helpers`:

~~~ ruby
<?php

namespace App\Helpers;

class ControllerHelper
{
    //
}
~~~

Make sure to register its location in the `aliases` array of `config/app.php`:

~~~ ruby
'ControllerHelper' => App\Helpers\ControllerHelper::class,
~~~

Our method to format the message should have three parameters: (1) the model, (2) title attribute of the model (in this case, but you can use other descriptive attributes) and (3) the operation performed.

~~~ ruby
public static function getNotificationMessage($model, $value, $method)
{
    return sprintf('%s "%s" %s', class_baseName($model), $value, $method);
}
~~~

> **NOTE**: [`class_baseName($class)`](http://stackoverflow.com/a/26296283) will provide the base name of the class.

Now we can simply use it in the controller as:

~~~ ruby
use ControllerHelper;

...

return redirect('daylogs')->with('message',
    ControllerHelper::getNotificationMessage(Daylog::class, $request->title, 'created'));
~~~

#### Check tests

Finally make sure that all tests pass:

~~~ shell
> ./vendor/bin/phpunit
PHPUnit 5.7.16 by Sebastian Bergmann and contributors.

...............                                                   15 / 15 (100%)

Time: 18.3 seconds, Memory: 19.25MB

OK (15 tests, 62 assertions)
~~~

### Improvements

* The original plan for `Task`s is to manage them within the create and edit views of `Daylog`s wherein they are dynamically added, edited or deleted.

    Here are screenshots of my previous project (before this one) called ["`laravel-daylog`"](https://bitbucket.org/giltroymeren/laravel-day-log) which was a combination of the [basic](https://laravel.com/docs/5.2/quickstart) and [intermediate](https://laravel.com/docs/5.2/quickstart-intermediate) Laravel tutorials.

    ![Concept for Daylog create view with dynamic Task - Initial]s/images/posts/2017-03-20-day-log-app/improvements/create-daylog-with-task-initial.png){:class="img-responsive"}

    ![Concept for Daylog create view with dynamic Task - Set]s/images/posts/2017-03-20-day-log-app/improvements/create-daylog-with-task-set.png){:class="img-responsive"}

    However I prioritized creating the application based of Flynsarmy and Barnes' original structure to serve as a foundation.

    I might implement this type of `Task` creation in this project. My current challenge with the other project is how to handle the edited `Task`s in the back-end because I need to keep track of both existing and newly-created ones. The latter are easy - they are just created but the first can either be changed or deleted during editing.

    I know I can just delete all `Task`s of the `Daylog` during editing (so I will just create all the `Task`s, no need to worry if they were changed or deleted) but I do not want to rely on such way. I have this mindset that in the future, these models will require a history or auditing system which means I need to keep tabs on which were actually changed or deleted.

* The view headers for `Daylog` and `Task` are currently simple and italicized. These can be further improved with another way of showing them with better design.

* The time form fields are currently simple textboxes. A date-time picker can be attached to these fields for better user experience.

* `Daylog`s and `Task`s are only deleted in singles. A select feature can make the User's management of these objects more convenient.

## References

* Barnes, Eric L. "Step by Step Guide to Building Your First Laravel Application." *Laravel News*. N.p., 05 Mar. 2016. Web. 12 Mar. 2017. \<[`https://laravel-news.com/your-first-laravel-application`](https://laravel-news.com/your-first-laravel-application)\>
* "Compact." *Php*. N.p., n.d. Web. 18 Mar. 2017. \<[`http://php.net/manual/en/function.compact.php`](http://php.net/manual/en/function.compact.php)\>.
* "Creating a Basic ToDo Application in Laravel 5 - Part 1." Flynsarmy. N.p., 13 May 2015. Web. 12 Mar. 2017. \<[`https://www.flynsarmy.com/2015/02/creating-a-basic-todo-application-in-laravel-5-part-1/`](https://www.flynsarmy.com/2015/02/creating-a-basic-todo-application-in-laravel-5-part-1/)\>
* "Creating a Basic ToDo Application in Laravel 5 - Part 2." Flynsarmy. N.p., 07 Aug. 2015. Web. 12 Mar. 2017. \<[`https://www.flynsarmy.com/2015/02/creating-a-basic-todo-application-in-laravel-5-part-2/`](https://www.flynsarmy.com/2015/02/creating-a-basic-todo-application-in-laravel-5-part-2/)\>
* "Creating a Basic ToDo Application in Laravel 5 - Part 3." Flynsarmy. N.p., 01 Apr. 2015. Web. 12 Mar. 2017. \<[`https://www.flynsarmy.com/2015/02/creating-a-basic-todo-application-in-laravel-5-part-3/`](https://www.flynsarmy.com/2015/02/creating-a-basic-todo-application-in-laravel-5-part-3/)\>
* "Creating a Basic ToDo Application in Laravel 5 - Part 4." Flynsarmy. N.p., 28 Mar. 2015. Web. 12 Mar. 2017. \<[`https://www.flynsarmy.com/2015/02/creating-a-basic-todo-application-in-laravel-5-part-4/`](https://www.flynsarmy.com/2015/02/creating-a-basic-todo-application-in-laravel-5-part-4/)\>
* "Forms & HTML." *Laravel Collective*. N.p., n.d. Web. 15 Mar. 2017. \<[`https://laravelcollective.com/docs/5.3/html`](https://laravelcollective.com/docs/5.3/html)\>.
* Heisian. "Best practices for custom helpers on Laravel 5." *Php - Best practices for custom helpers on Laravel 5 - Stack Overflow*. N.p., 24 Sept. 2015. Web. 02 Apr. 2017. \<[`http://stackoverflow.com/a/32772686`](http://stackoverflow.com/a/32772686)\>.
* Otwell, Taylor. "Basic Task List." *Basic Task List - Laravel - The PHP Framework For Web Artisans*. N.p., n.d. Web. 18 Mar. 2017. \<[`https://laravel.com/docs/5.2/quickstart`](https://laravel.com/docs/5.2/quickstart)\>
* Otwell, Taylor. "Browser Tests (Laravel Dusk)." *Browser Tests (Laravel Dusk) - Laravel - The PHP Framework For Web Artisans*. N.p., n.d. Web. 15 May 2017. \<[`https://laravel.com/docs/5.4/dusk`](https://laravel.com/docs/5.4/dusk)\>.
* Otwell, Taylor. "Intermediate Task List." *Intermediate Task List - Laravel - The PHP Framework For Web Artisans*. N.p., n.d. Web. 02 Mar. 2017. \<[`https://laravel.com/docs/5.2/quickstart-intermediate`](https://laravel.com/docs/5.2/quickstart-intermediate)\>
* Otwell, Taylor. "Routing." *Routing - Laravel - The PHP Framework For Web Artisans*. N.p., n.d. Web. 15 Mar. 2017. \<[`https://laravel.com/docs/5.4/routing#route-model-binding`](https://laravel.com/docs/5.4/routing#route-model-binding)\>.
* Otwell, Taylor. "Validation." *Validation - Laravel - The PHP Framework For Web Artisans*. N.p., n.d. Web. 12 Mar. 2017. \<[`https://laravel.com/docs/5.4/validation`](https://laravel.com/docs/5.4/validation)\>.
* Raza, Shoaib. "Laravel 5 eloquent model auto add an underscore." *Php - Laravel 5 eloquent model auto add an underscore - Stack Overflow*. N.p., 14 Jan. 2016. Web. 18 Mar. 2017. \<[`http://stackoverflow.com/a/34781062`](http://stackoverflow.com/a/34781062)\>.
* Tkaczyk, Jarek. "Laravel get class name of related model." *Php - Laravel get class name of related model - Stack Overflow*. N.p., 10 Oct. 2014. Web. 02 Apr. 2017. \<[`http://stackoverflow.com/a/26296283`](http://stackoverflow.com/a/26296283)\>.
* Zaninotto, Francois. "Fzaninotto/Faker." *GitHub*. N.p., 15 Oct. 2011. Web. 12 Mar. 2017. \<[`https://github.com/fzaninotto/Faker`](https://github.com/fzaninotto/Faker)\>.

## Changes

* `2017-04-02`
    * Added "Changes" section.
    * Added "[Check tests](#check-tests)" subsection.
    * Moved actual guide under single header to separate from other subtopics.
    * Updated headers' organization.
    * Added "[Create a helper class for notification messages](#create-a-helper-class-for-notification-messages)" subsection.