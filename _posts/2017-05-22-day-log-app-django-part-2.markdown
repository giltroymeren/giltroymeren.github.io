---
layout: post
title:  "Day Log App in Django Part 2: View DayLog home page"
date:   2017-05-22
categories: projects notes python django series daylogapp
---

This post will discuss how to implement the front page where a table of available day logs will be shown.

## Table of Contents
* TOC
{:toc}

## Steps

### Create test

Update `daylog/test.py` with the test to check the return value of the view method responsible for the home page:

~~~ python
from django.core.urlresolvers import reverse
from django.test import TestCase

from .models import Daylog

...

class ViewTests(TestCase):
    def test_index_with_no_daylogs(self):
        response = self.client.get(reverse('daylog:get_list'))
        self.assertEqual(response.status_code, 200)
        self.assertContains(response, "There are no Day Logs available.")
        self.assertQuerysetEqual(response.context['daylogs'], [])

    def test_index_view_with_a_daylog(self):
        create_daylog()
        response = self.client.get(reverse('daylog:get_list'))

        self.assertQuerysetEqual(
            response.context['daylogs'],
            ['<Daylog: 2017-01-01: Lorem ipsum dolor sit amet>']
        )
~~~

The method `create_daylog()` will return a `Daylog` instance:

~~~ python
def create_daylog():
    return Daylog.objects.create(
        title = "Lorem ipsum dolor sit amet",
        slug = "lorem-ipsum",
        location = "ABC XYZ-112233",
        log_at = "2017-01-01",
        category = Daylog.CATEGORY_ADEQUATE,
    )
~~~

Finally do not forget to update the `ModelTests` class:

~~~ python
class ModelTests(TestCase):
    def test_model_can_create_a_daylog(self):
        old_count = Daylog.objects.count()
        create_daylog().save()
        new_count = Daylog.objects.count()

        self.assertTrue(old_count < new_count)
~~~

### Set URLs

There are two file that need to be updated to make sure that the URL `http://127.0.0.1:8000/daylog/` (in my local environment) points to the correct view method:

*   `api/urls.py`

    As said before, the `api` will handle the applications created in the project and it is `daylog` in this context.

    Add the URL path that starts with `daylog` to point to the `urls.py` of the `daylog` app:

    ~~~ python
    from django.conf.urls import url, include
    ...
    urlpatterns = [
        url(r'^admin/', admin.site.urls),
        url(r'^daylog/', include('daylog.urls')),
    ~~~

*   `daylog/urls.py`

    Create this file.

    All URLs in this file are be automatically preceeded by `daylog` as set in `api/urls.py`.

    ~~~ python
    from django.conf.urls import url, include

    from . import views

    app_name = 'daylog'
    urlpatterns = [
        url(r'^S', views.get_list, name='get_list'),
    ]
    ~~~

    The path `daylog/` is now mapped to the alias and method `get_list` in the `daylog/views.py`.

### Create view method

One of the peculiar aspects of Django I found, coming from Laravel, Spring MVC and Node, is that the controllers are actually the methods in the view file (meaning `views.py`) and the UI view files are templates (usually found as `app-name/templates/app-name/index.html`).

Add the first method in `daylog/views.py`:

~~~ python
from django.shortcuts import render

from .models import Daylog

def get_list(request):
    daylogs = Daylog.objects.all()
    return render(request, 'daylog/index.html', { 'daylogs': daylogs })
~~~

This will need the template file `index.html` that must be created under a new folder `templates/daylog`. Specifically the path is `daylog/templates/daylog/index.html` because Django needs to resolve the pathname of the app under `templates` as well hence the same folder name as the app.

### Create template files

#### Base template

Create `daylog/templates/daylog/base.html`:

~~~ html
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="utf-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1">

        <title>DayLogApp</title>

        <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0-alpha.6/css/bootstrap.min.css" integrity="sha384-rwoIResjU2yc3z8GV/NPeZWAv56rSmLldC3R/AZzGRnGxQQKnKkoFVhFQhNUwEyJ" crossorigin="anonymous">
    </head>

    <body>

        <nav class="navbar navbar-toggleable-md navbar-light bg-faded" style="margin-bottom: 2em">
            <div class="container col-sm-6">
                <button class="navbar-toggler navbar-toggler-right" type="button"
                    data-toggle="collapse" data-target="#navbar"
                    aria-controls="navbar" aria-expanded="false" aria-label="Toggle navigation">
                    <span class="navbar-toggler-icon"></span>
                </button>
                <a class="navbar-brand" href="/">DayLogApp</a>
            </div>
        </nav>

        <div class="row">
            <div class="container col-sm-6">
            {% raw %}
            {% block content %}
            {% endblock %}
            {% endraw %}
            </div>
        </div>

        <script src="https://code.jquery.com/jquery-3.1.1.slim.min.js" integrity="sha384-A7FZj7v+d/sdmMqp/nOQwliLvUsJfDHW+k9Omg/a/EheAdgtzNs3hpfag6Ed950n" crossorigin="anonymous"></script>
        <script src="https://cdnjs.cloudflare.com/ajax/libs/tether/1.4.0/js/tether.min.js" integrity="sha384-DztdAPBWPRXSA/3eYEEUWrWCy7G5KFbe8fFjk5JAIxUYHKkDx6Qin1DkWx51bBrb" crossorigin="anonymous"></script>
        <script src="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0-alpha.6/js/bootstrap.min.js" integrity="sha384-vBWWzlZJ8ea9aCX4pEW3rVHjgjt7zpkNpZk+02D9phzyeVkE+jo0ieGizqPLForn" crossorigin="anonymous"></script>

    </body>
</html>
~~~

#### Home page

~~~ html
{% raw %}
{% extends 'daylog/base.html' %}

{% block content %}

    <div class="card">
        <h3 class="card-header">Day Log</h3>

        {% if daylogs %}
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
            {% for daylog in daylogs %}
                <tr>
                    <td>
                        <a href="/daylog/{{ daylog.slug }}">
                            <strong>{{ daylog.title }}</strong></td>
                        </a>
                    <td>
                        <code>{{ daylog.created_at|date:"Y-m-d" }}</a>
                        </code>
                    </td>
                    <td>
                        <code>{{ daylog.modified_at|date:"Y-m-d" }}</code>
                    </td>
                    <td><code>0</code></td>
                    <td>
                        <a href="/daylog/{{ daylog.slug }}/edit" class="btn btn-primary btn-sm">Edit</a>
                    </td>
                    <td>
                        <form action="/daylog/{{ daylog.slug }}/delete" method="POST">
                            <input type='hidden' value='DELETE' name='_method'>
                            <button type="submit" class="btn btn-sm btn-danger">Delete</button>
                        </form>
                    </td>
                </tr>
            {% endfor %}
            </tbody>
        </table>
        {% else %}
        <div class="card-block">
            <div class="alert alert-info" role="alert">
                There are no Day Logs available.
            </div>
        </div>
        {% endif %}

        <div class="card-footer text-center">
            <a href="/daylog/create" class=" btn btn-primary">Create Day Log</a>
        </div>

    </div><!-- .card -->

{% endblock %}
{% endraw %}
~~~

Please refer to the [Django Templates](https://docs.djangoproject.com/en/1.11/topics/templates/) document for more information.

![Home page with one Daylog](/images/posts/2017-05-22-day-log-app-django-part-2/table-not-empty.png){:class="img-responsive"}

![Home page with one Daylog](/images/posts/2017-05-22-day-log-app-django-part-2/table-empty.png){:class="img-responsive"}

### Run tests

The two new test cases should pass:

~~~ console
$ python manage.py test
Creating test database for alias 'default'...
...
----------------------------------------------------------------------
Ran 3 tests in 0.075s

OK
Destroying test database for alias 'default'...
~~~

### Running the application

~~~ console
$ python manage.py runserver
Performing system checks...

System check identified no issues (0 silenced).
May 22, 2017 - 10:28:10
Django version 1.9.4, using settings 'api.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
~~~

### Manually add `Daylog`s into the database

Right now there is no way in the application to add a `Daylog` to test the view. The Python shell can be used to mainly experiment with the application's features including the model and database.

Below is a sample of creating a new `Daylog` called "Lorem ipsum" through the shell:

~~~ python
$ python manage.py shell
Python 3.6.1 (v3.6.1:69c0db5050, Mar 21 2017, 01:21:04)
...
Type "help", "copyright", "credits" or "license" for more information.
(InteractiveConsole)
>>> from daylog.models import Daylog
>>> Daylog.objects.create(title="Lorem ipsum",slug="lorem-ipsum",location="ABC XYZ-1010 QWERTY",log_at="2016-01-01",category="Minor")
<Daylog: 2016-01-01: Lorem ipsum>
~~~

## References
* "Documentation." *Writing your first Django app, part 1 &#124; Django documentation &#124; Django*. N.p., n.d. Web. 19 May 2017. <[`https://docs.djangoproject.com/en/1.11/intro/tutorial01/`](https://docs.djangoproject.com/en/1.11/intro/tutorial01/)>.
* "Documentation." *Writing your first Django app, part 2 &#124; Django documentation &#124; Django*. N.p., n.d. Web. 19 May 2017. <[`https://docs.djangoproject.com/en/1.11/intro/tutorial02/`](https://docs.djangoproject.com/en/1.11/intro/tutorial02/)>.
* "Documentation." *Writing your first Django app, part 3 &#124; Django documentation &#124; Django*. N.p., n.d. Web. 19 May 2017. <[`https://docs.djangoproject.com/en/1.11/intro/tutorial03/`](https://docs.djangoproject.com/en/1.11/intro/tutorial03/)>.
* "Documentation." *Writing your first Django app, part 4 &#124; Django documentation &#124; Django*. N.p., n.d. Web. 19 May 2017. <[`https://docs.djangoproject.com/en/1.11/intro/tutorial04/`](https://docs.djangoproject.com/en/1.11/intro/tutorial04/)>.
* "Documentation." *Writing your first Django app, part 5 &#124; Django documentation &#124; Django*. N.p., n.d. Web. 19 May 2017. <[`https://docs.djangoproject.com/en/1.11/intro/tutorial04/`](https://docs.djangoproject.com/en/1.11/intro/tutorial05/)>.
* Freitas, Vitor. "How to Implement CRUD Using Ajax and Json." *Simple is Better Than Complex*. Simple is Better Than Complex, 15 Nov. 2016. Web. 19 May 2017. <[`https://simpleisbetterthancomplex.com/tutorial/2016/11/15/how-to-implement-a-crud-using-ajax-and-json.html`](https://simpleisbetterthancomplex.com/tutorial/2016/11/15/how-to-implement-a-crud-using-ajax-and-json.html)>.
* Gikera, Jee. "Build a REST API with Django – A Test Driven Approach: Part 1." *Scotch*. N.p., 6 Feb. 2017. Web. 19 May 2017. <[`https://scotch.io/tutorials/build-a-rest-api-with-django-a-test-driven-approach-part-1`](https://scotch.io/tutorials/build-a-rest-api-with-django-a-test-driven-approach-part-1)>.
