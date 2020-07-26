---
layout: post
title:  "Day Log App in Django Part 3: Create a DayLog"
date:   2017-05-23
categories: projects notes python django series daylogapp
---

This post will discuss how to implement the creation of a `Daylog` through a form.

## Table of Contents
* TOC
{:toc}

## Steps

### Show the form

#### Create test

Add the following in `daylog/tests/py`:

~~~ python
def test_create_view(self):
    response = self.client.get(reverse('daylog:create'))

    self.assertEqual(response.context['action'], 'daylog:store')
    self.assertEqual(response.context['operation'], 'create')
~~~

#### Set URL

`daylog/create` is the path to the form page with alias and view method `create`.

~~~ python
url(r'^create/$', views.create, name='create'),
~~~

#### Create create method

~~~ python
def create(request):
    return render(request, 'daylog/form.html', {
        'action': 'daylog:create',
        'operation': 'create',
    })
~~~

#### Create page

~~~ html
{% raw %}
{% extends 'daylog/base.html' %}

{% block content %}
<div class="card">
    <h3 class="card-header">{{ operation | capfirst }} Day Log</h3>

    <div class="card-block">

        <form action="{% if action %}{% url action %}{% endif %}" method="post"
            enctype="application/x-www-form-urlencoded">
            {% csrf_token %}

            {% if operation == 'update' %}
                <input type="hidden" value="PUT" name="_method">
            {% endif %}

            <div class="container">
                <div class="form-group row">
                    <label for="title" class="col-sm-2 col-form-label text-right">Title</label>
                    <div class="col-sm-10">
                        <textarea class="form-control" id="title" name="title"
                            placeholder="Describe your log" required
                            {% if operation == 'view' %}readonly{% endif %}
                            >{% if daylog %}{{ daylog.title }}{% endif %}</textarea>
                    </div>
                </div>

                <div class="form-group row">
                    <label for="slug" class="col-sm-2 col-form-label text-right">Slug</label>
                    <div class="col-sm-10">
                        <input type="text" class="form-control" id="slug" name="slug"
                            placeholder="example-daylog-slug" required
                            {% if operation == 'view' or operation == 'update' %}readonly{% endif %}
                            value="{% if daylog %}{{ daylog.slug }}{% endif %}">
                    </div>
                </div>

                <div class="form-group row">
                    <label for="location" class="col-sm-2 col-form-label text-right">Location</label>
                    <div class="col-sm-10">
                        <input type="text" class="form-control" id="location" name="location"
                            placeholder="Location of log" required
                            {% if operation == 'view' %}readonly{% endif %}
                            value="{% if daylog %}{{ daylog.location }}{% endif %}">
                    </div>
                </div>

                <div class="form-group row">
                    <label for="log_at" class="col-sm-2 col-form-label text-right">Date of log</label>
                    <div class="col-sm-10">
                        <input type="date" class="form-control" id="log_at" name="log_at"
                            placeholder="Date of log" required
                            {% if operation == 'view' %}readonly{% endif %}
                            value="{% if daylog %}{{ daylog.log_at  }}{% endif %}">
                    </div>
                </div>

                <div class="form-group row">
                    <label class="col-sm-2 col-form-label text-right">Category</label>
                    <div class="col-sm-10">
                        <div class="form-check">
                            <label class="form-check-label">
                                <input class="form-check-input" type="radio"
                                    name="category" id="category" value="Adequate" required
                                    {% if operation == 'view' %}disabled{% endif %}
                                    {% if daylog and daylog.category == "Adequate" %}checked{% endif %}>
                                Adequate
                            </label>
                        </div>
                        <div class="form-check">
                            <label class="form-check-label">
                                <input class="form-check-input" type="radio"
                                    name="category" id="category" value="Minor" required
                                    {% if operation == 'view' %}disabled{% endif %}
                                    {% if daylog and daylog.category == "Minor" %}checked{% endif %}>
                                Minor
                            </label>
                        </div>
                        <div class="form-check">
                            <label class="form-check-label">
                                <input class="form-check-input" type="radio"
                                    name="category" id="category" value="Major" required
                                    {% if operation == 'view' %}disabled{% endif %}
                                    {% if daylog and daylog.category == "Major" %}checked{% endif %}>
                                Major
                            </label>
                        </div>
                    </div>
                </div>

                {% if operation != 'view' %}
                <div class="form-group row">
                    <div class="offset-sm-2 col-sm-10">
                        <button type="submit" class="btn btn-primary">Submit</button>
                        <button type="reset" class="btn btn-danger">Reset</button>
                    </div>
                </div>
                {% endif %}
            </div><!-- .container -->

        </form>

    </div><!-- .card-block -->

    <div class="card-footer text-center">
        <a href="/daylog" class=" btn btn-primary btn-sm">Back</a>
    </div>

</div><!-- .card -->

{% endblock %}
{% endraw %}
~~~

`test_create_view()` should pass after.

### Submit the form

#### Create test

Three test cases will check for: valid, empty and invalid forms submitted.

First, `create_daylog()` is updated to use a dictionary of raw `Daylog` data for reuse.
~~~ python
def get_daylog_data():
    return {
        "title": "Lorem ipsum dolor sit amet",
        "slug": "lorem-ipsum",
        "location": "ABC XYZ-112233",
        "log_at": "2017-01-01",
        "category": Daylog.CATEGORY_ADEQUATE,
    }

def create_daylog():
    daylog = get_daylog_data()
    return Daylog.objects.create(
        title = daylog["title"],
        slug = daylog["slug"],
        location = daylog["location"],
        log_at = daylog["log_at"],
        category = daylog["category"],
    )
~~~

*   Valid `Daylog`

    ~~~ python
    from django.test import Client, TestCase

    ...

    def test_create_and_submit_with_valid_daylog(self):
        test_daylog = get_daylog_data()
        response = self.client.post(reverse('daylog:create'), test_daylog, follow=True)

        db_daylog = Daylog.objects.get(slug = test_daylog['slug'])

        self.assertEqual(test_daylog['title'], db_daylog.title)
        self.assertEqual(test_daylog['slug'], db_daylog.slug)
        self.assertEqual(test_daylog['location'], db_daylog.location)
        self.assertEqual(test_daylog['log_at'], db_daylog.log_at.strftime('%Y-%m-%d'))
        self.assertEqual(test_daylog['category'], db_daylog.category)

        for message in response.context['messages']:
            self.assertEqual(message.message, "Day Log created.")
            self.assertEqual(message.extra_tags, "info")
            break;
    ~~~

*   Empty `Daylog`

    ~~~ python
    def test_create_and_submit_with_empty_daylog(self):
        response = self.client.post(reverse('daylog:create'), {})

        self.assertTrue("title" in response.context['errors'])
        self.assertTrue("slug" in response.context['errors'])
        self.assertTrue("location" in response.context['errors'])
        self.assertTrue("log_at" in response.context['errors'])
        self.assertTrue("category" in response.context['errors'])
    ~~~

*   Invalid `Daylog`

    ~~~ python
    def test_create_and_submit_with_existing_slug_and_log_at(self):
        create_daylog()

        test_daylog = get_daylog_data()
        response = self.client.post(reverse('daylog:create'), test_daylog)

        self.assertTrue("slug" in response.context['errors'])
        self.assertTrue("log_at" in response.context['errors'])
    ~~~

In the valid and invalid test cases, the [Django flash messages](https://docs.djangoproject.com/en/1.10/ref/contrib/messages/) are used to validate the method. This is under the `django.contrib` library.

#### Update create method

We add a catch for `POST` requests in the `create` method to contain the actual processing:

~~~ python
from django.contrib import messages

from .forms import DaylogForm
...
def create(request):
    if request.method == 'POST':
        daylog_form = DaylogForm(request.POST)
        if daylog_form.is_valid():
            daylog_form.save()
            messages.info(request, "Day Log created.", "info")
            return get_list(request)
        else:
            return render(request, 'daylog/form.html', {
                'daylog': request.POST,
                'errors': daylog_form.errors
             })

    return render(request, 'daylog/form.html', {
        'action': 'daylog:create',
        'operation': 'create',
    })
~~~

Django has an available model-form mapping system class called [`ModelForm`](https://docs.djangoproject.com/en/1.11/topics/forms/modelforms/#modelform). This is used for the validation of fields.

Create `daylog/forms.py`:

~~~ python
from django import forms

from .models import Daylog

class DaylogForm(forms.ModelForm):
    class Meta:
        model = Daylog
        fields = "__all__"
~~~

The tests should pass after.

### Standardize date format

I wanted to set all values of `log_at` to use the format `YYYY-MM-DD`. In order to set this globally for `models.DateField()` instances, just add these in `api/settings.py` in the "Internalization" section:

~~~ python
USE_L10N = False
...
DATE_FORMAT = 'Y-m-d'
~~~

## References
* "Documentation." *The messages framework &#124; Django documentation &#124; Django*. N.p., n.d. Web. 21 May 2017. <[`https://docs.djangoproject.com/en/1.10/ref/contrib/messages/`](https://docs.djangoproject.com/en/1.10/ref/contrib/messages/)>.
* "Documentation." *Writing your first Django app, part 1 &#124; Django documentation &#124; Django*. N.p., n.d. Web. 19 May 2017. <[`https://docs.djangoproject.com/en/1.11/intro/tutorial01/`](https://docs.djangoproject.com/en/1.11/intro/tutorial01/)>.
* "Documentation." *Writing your first Django app, part 2 &#124; Django documentation &#124; Django*. N.p., n.d. Web. 19 May 2017. <[`https://docs.djangoproject.com/en/1.11/intro/tutorial02/`](https://docs.djangoproject.com/en/1.11/intro/tutorial02/)>.
* "Documentation." *Writing your first Django app, part 3 &#124; Django documentation &#124; Django*. N.p., n.d. Web. 19 May 2017. <[`https://docs.djangoproject.com/en/1.11/intro/tutorial03/`](https://docs.djangoproject.com/en/1.11/intro/tutorial03/)>.
* "Documentation." *Writing your first Django app, part 4 &#124; Django documentation &#124; Django*. N.p., n.d. Web. 19 May 2017. <[`https://docs.djangoproject.com/en/1.11/intro/tutorial04/`](https://docs.djangoproject.com/en/1.11/intro/tutorial04/)>.
* "Documentation." *Writing your first Django app, part 5 &#124; Django documentation &#124; Django*. N.p., n.d. Web. 19 May 2017. <[`https://docs.djangoproject.com/en/1.11/intro/tutorial04/`](https://docs.djangoproject.com/en/1.11/intro/tutorial05/)>.
* Dunatotatos. "How can I unit test django messages?" *Python - How can I unit test django messages? - Stack Overflow*. N.p., 16 Feb. 2013. Web. 21 May 2017. <[`http://stackoverflow.com/questions/2897609/how-can-i-unit-test-django-messages#comment72873647_14909727`](http://stackoverflow.com/questions/2897609/how-can-i-unit-test-django-messages#comment72873647_14909727)>.
* Freitas, Vitor. "How to Implement CRUD Using Ajax and Json." *Simple is Better Than Complex*. Simple is Better Than Complex, 15 Nov. 2016. Web. 19 May 2017. <[`https://simpleisbetterthancomplex.com/tutorial/2016/11/15/how-to-implement-a-crud-using-ajax-and-json.html`](https://simpleisbetterthancomplex.com/tutorial/2016/11/15/how-to-implement-a-crud-using-ajax-and-json.html)>.
* Gikera, Jee. "Build a REST API with Django – A Test Driven Approach: Part 1." *Scotch*. N.p., 6 Feb. 2017. Web. 19 May 2017. <[`https://scotch.io/tutorials/build-a-rest-api-with-django-a-test-driven-approach-part-1`](https://scotch.io/tutorials/build-a-rest-api-with-django-a-test-driven-approach-part-1)>.
* Nicol, Alasdair. "Django Testing - check messages for a view that redirects." *Stack Overflow*. N.p., 22 Apr. 2013. Web. 21 May 2017. <[`https://stackoverflow.com/a/16143864`](https://stackoverflow.com/a/16143864)>.
* Roberts, Ben. "Django Template Ternary Operator." *Python - Django Template Ternary Operator - Stack Overflow*. N.p., 27 Apr. 2013. Web. 20 May 2017. <[`https://stackoverflow.com/questions/3110166/django-template-ternary-operator#comment23256292_3110218`](https://stackoverflow.com/questions/3110166/django-template-ternary-operator#comment23256292_3110218)>.
* Turikumwe, Jean Paul. "How to break "for loop" in Django template." *How to break "for loop" in Django template - Stack Overflow*. N.p., 7 Nov. 2011. Web. 22 May 2017. <[`http://stackoverflow.com/a/8036551`](http://stackoverflow.com/a/8036551)>.
* Vazquez-Abrams, Ignacio . "Django template date format." *Stack Overflow*. N.p., 12 Oct. 2011. Web. 21 May 2017. <[`http://stackoverflow.com/a/7737172`](http://stackoverflow.com/a/7737172)>.
