---
layout: post
title:  "Day Log App in Django Part 1: Day Log model"
date:   2017-05-22
categories: projects notes python django series daylogapp
---

This is a [Django](https://www.djangoproject.com/) implementation of the [original Laravel application]({{ site.baseurl }}{% post_url 2017-03-20-day-log-app %}).

The scope of this post is from creation of the project to the [test-driven development](https://en.wikipedia.org/wiki/Test-driven_development) of the `Daylog` model.

## Table of Contents
* TOC
{:toc}

## Steps

Please refer to the [official tutorial](https://docs.djangoproject.com/en/1.11/intro/tutorial01/) for the necessary applications to start Django development.

### Create project

Make a folder to contain the project and go inside it. I chose `daylogapp-django`.

Create the Django project with:

    django-admin startproject api

`api` is the name of the project and the command should have the following structure:

~~~
daylogapp-django/
└── api*/
    ├── api/
    │   ├── __init__.py
    │   ├── settings.py
    │   ├── urls.py
    │   └── wsgi.py
    └── manage.py
~~~

> **NOTE**: *In virtue of the original project I renamed the topmost `api` folder to `daylogapp`. This will not affect other parts of the project.

Django can have many applications inside that can interact with each other and this initial project `api` will be the "central command". New apps of the project will be registered in this `api` that will be discussed later.

### Create `daylog` app

Go to the `daylogapp` folder with the file `manage.py` and run:

    python manage.py startapp daylog

The folder `daylog` should be present after. Now we have two "apps" - `api` and `daylog`.

Next, go to `api/settings.py` to register our new app in the `INSTALLED_APPS` variable together with the [Django Rest Framework](http://www.django-rest-framework.org/):

~~~ python
    'rest_framework',

    'daylog',
~~~

### Create `Daylog` model

#### Create test case

Go to `daylog/tests.py` and add the following:

~~~ python
from django.test import TestCase

from .models import Daylog

class ModelTests(TestCase):

    def setUp(self):
        self.daylog_title = "Lorem ipsum dolor sit amet"
        self.daylog_slug = "lorem-ipsum"
        self.daylog_location = "ABC XYZ-112233"
        self.daylog_log_at = "2017-01-01"
        self.daylog_category = Daylog.CATEGORY_ADEQUATE

        self.daylog = Daylog(
            title = self.daylog_title,
            slug = self.daylog_slug,
            location = self.daylog_location,
            log_at = self.daylog_log_at,
            category = self.daylog_category,
        )

    def test_model_can_create_a_daylog(self):
        old_count = Daylog.objects.count()
        self.daylog.save()
        new_count = Daylog.objects.count()

        self.assertTrue(old_count < new_count)
~~~

The test will check if a `Daylog` instance is actually created after saving.

If we run the test, we should expect that it will fail:

~~~ console
$ python manage.py test
Creating test database for alias 'default'...
E
======================================================================
ERROR: daylog.tests (unittest.loader._FailedTest)
----------------------------------------------------------------------
ImportError: Failed to import test module: daylog.tests
Traceback (most recent call last):
  File "/Library/Frameworks/Python.framework/Versions/3.6/lib/python3.6/unittest/loader.py", line 428, in _find_test_path
    module = self._get_module_from_name(name)
  File "/Library/Frameworks/Python.framework/Versions/3.6/lib/python3.6/unittest/loader.py", line 369, in _get_module_from_name
    __import__(name)
  File "/path/to/daylogapp-node/daylogapp/daylog/tests.py", line 3, in <module>
    from .models import Daylog
ImportError: cannot import name 'Daylog'


----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (errors=1)
Destroying test database for alias 'default'...
~~~

#### Define model structure

The structure of the `Daylog` is the same as the previous implementations. Go to `daylog/models.py` and add:

~~~ python
from django.db import models

class Daylog(models.Model):
    CATEGORY_ADEQUATE = "Adequate"
    CATEGORY_MINOR = "Minor"
    CATEGORY_MAJOR = "Major"
    CATEGORY_CHOICES = (
        (CATEGORY_ADEQUATE, CATEGORY_ADEQUATE),
        (CATEGORY_MINOR, CATEGORY_MINOR),
        (CATEGORY_MAJOR, CATEGORY_MAJOR),
    )

    title = models.CharField(max_length=255, blank=False)
    slug = models.CharField(max_length=50, blank=False, unique=True)
    location = models.CharField(max_length=100, blank=False)
    log_at = models.DateField(blank=False, unique=True)
    category = models.CharField(max_length=25, choices=CATEGORY_CHOICES, blank=False)

    created_at = models.DateTimeField(auto_now_add=True)
    modified_at = models.DateTimeField(auto_now=True)

    def __str__(self):
        return "%s: %s" % (self.log_at, self.title)
~~~

`CATEGORY_CHOICES` is just a constant attribute and the `__str__` method was overriden to expect a string representation similar to `<Daylog: 2016-01-01: Lorem ipsum>`.

#### Migrate tables

There are two succeeding commands required whenever any changes in the models are made: `makemigrations` and `migrate`.

~~~ console
$ python manage.py makemigrations
Migrations for 'daylog':
  0001_initial.py:
    - Create model Daylog
~~~

~~~ console
$ python manage.py migrate
Operations to perform:
  Apply all migrations: auth, contenttypes, sessions, admin, daylog
Running migrations:
  Rendering model states... DONE
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying daylog.0001_initial... OK
  Applying sessions.0001_initial... OK
~~~

#### Rerun tests

The model test case should pass:

~~~ console
$ python manage.py test
Creating test database for alias 'default'...
.
----------------------------------------------------------------------
Ran 1 test in 0.002s

OK
Destroying test database for alias 'default'...
~~~

## References
* "Documentation." *Writing your first Django app, part 1 &#124; Django documentation &#124; Django*. N.p., n.d. Web. 19 May 2017. <[`https://docs.djangoproject.com/en/1.11/intro/tutorial01/`](https://docs.djangoproject.com/en/1.11/intro/tutorial01/)>.
* "Documentation." *Writing your first Django app, part 2 &#124; Django documentation &#124; Django*. N.p., n.d. Web. 19 May 2017. <[`https://docs.djangoproject.com/en/1.11/intro/tutorial02/`](https://docs.djangoproject.com/en/1.11/intro/tutorial02/)>.
* "Documentation." *Writing your first Django app, part 5 &#124; Django documentation &#124; Django*. N.p., n.d. Web. 19 May 2017. <[`https://docs.djangoproject.com/en/1.11/intro/tutorial04/`](https://docs.djangoproject.com/en/1.11/intro/tutorial05/)>.
* Freitas, Vitor. "How to Implement CRUD Using Ajax and Json." *Simple is Better Than Complex*. Simple is Better Than Complex, 15 Nov. 2016. Web. 19 May 2017. <[`https://simpleisbetterthancomplex.com/tutorial/2016/11/15/how-to-implement-a-crud-using-ajax-and-json.html`](https://simpleisbetterthancomplex.com/tutorial/2016/11/15/how-to-implement-a-crud-using-ajax-and-json.html)>.
* Gikera, Jee. "Build a REST API with Django – A Test Driven Approach: Part 1." *Scotch*. N.p., 6 Feb. 2017. Web. 19 May 2017. <[`https://scotch.io/tutorials/build-a-rest-api-with-django-a-test-driven-approach-part-1`](https://scotch.io/tutorials/build-a-rest-api-with-django-a-test-driven-approach-part-1)>.
