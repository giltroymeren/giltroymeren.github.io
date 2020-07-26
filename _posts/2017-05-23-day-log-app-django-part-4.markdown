---
layout: post
title:  "Day Log App in Django Part 4: Project view templates"
date:   2017-05-23
categories: projects notes python django series daylogapp
---

As the project grows there is a need to re-use template files across Django applications. This post will show a way to declare templates for global use in the project.

## Table of Contents
* TOC
{:toc}

## Steps

In the previous posts, templates are declared in the path `app_name/templates/app_name/index.html` and they are `extend`ed in other template files as `app_name/index.html`.

### Setup

The way in Django 1.8 and above is to simply:

1.  Declare a common directory path in the `DIRS` key

    Go to `api/settings.py` and there should be the `TEMPLATES` variable.

    I added the path below.

    ~~~ python
    'DIRS': [(os.path.join(BASE_DIR, 'templates')),],
    ~~~

2. Create the `templates` folder in the same level as all the applications.

### Refactor `base.html`

Currently `base.html` is inside the `daylog` application'. In the future, in case other applications need it, just transfer it inside the new `templates` folder and call in the other template files as:

~~~ html
{% raw %}
{% extends 'base.html' %}
{% endraw %}
~~~

### Setup alerts

Create the file `alert.html` in the new `templates` folder:

~~~ html
<div class="alert alert-{{ type }}" role="alert">
    <button type="button" class="close" data-dismiss="alert" aria-label="Close">
        <span aria-hidden="true">&times;</span>
    </button>
    {{ message | striptags }}
</div>
~~~

This layout follows the [Bootstrap 4 Alerts](https://v4-alpha.getbootstrap.com/components/alerts/) module. It expects `type` and `message` variables. A `striptags` filter was added in case the returned `message` has any.

#### `base.html`

There were Django flash messages used in the `views.py` methods that were set.

In `base.html`, in orderto show an alert for every CRUD process, add the following block before `{% raw %}{% block content %}{% endraw %}`.

~~~ python
{% raw %}
{% if messages %}
    {% for message in messages|slice:":1" %}
        {% include "alert.html" with type=message.extra_tags  message=message %}
    {% endfor %}
{% endif %}
{% endraw %}
~~~

This will expect a variable `messages` that will contain a single message (hence the `:1` `slice` filter).

![Home page with create alert](/images/posts/2017-05-23-day-log-app-django-part-4/daylog-create-alert.png){:class="img-responsive"}

Additionally the form validation errors from Django should be shown as well in case the HTML5 interface fails. In `templates/daylog/form.html`, each `errors` variable from the `ModelForm` is expected:

~~~ html
{% raw %}
{% if errors.title %}
    {% include "alert.html" with type='danger' message=errors.title %}
{% endif %}
{% endraw %}
~~~

![Create page with error alerts](/images/posts/2017-05-23-day-log-app-django-part-4/create-error-alerts.png){:class="img-responsive"}

## References
* "Documentation." *Writing your first Django app, part 3 &#124; Django documentation &#124; Django*. N.p., n.d. Web. 19 May 2017. <[`https://docs.djangoproject.com/en/1.11/intro/tutorial03/`](https://docs.djangoproject.com/en/1.11/intro/tutorial03/)>.
* "Documentation." *Writing your first Django app, part 4 &#124; Django documentation &#124; Django*. N.p., n.d. Web. 19 May 2017. <[`https://docs.djangoproject.com/en/1.11/intro/tutorial04/`](https://docs.djangoproject.com/en/1.11/intro/tutorial04/)>.
* Freitas, Vitor. "How to Implement CRUD Using Ajax and Json." *Simple is Better Than Complex*. Simple is Better Than Complex, 15 Nov. 2016. Web. 19 May 2017. <[`https://simpleisbetterthancomplex.com/tutorial/2016/11/15/how-to-implement-a-crud-using-ajax-and-json.html`](https://simpleisbetterthancomplex.com/tutorial/2016/11/15/how-to-implement-a-crud-using-ajax-and-json.html)>.
* Gikera, Jee. "Build a REST API with Django – A Test Driven Approach: Part 1." *Scotch*. N.p., 6 Feb. 2017. Web. 19 May 2017. <[`https://scotch.io/tutorials/build-a-rest-api-with-django-a-test-driven-approach-part-1`](https://scotch.io/tutorials/build-a-rest-api-with-django-a-test-driven-approach-part-1)>.
* Sosa, Oliver. "Base template for all apps in Django." *Python 2.7 - Base template for all apps in Django - Stack Overflow*. N.p., 16 Nov. 2016. Web. 22 May 2017. <[`https://stackoverflow.com/a/40642864`](https://stackoverflow.com/a/40642864)>.
* Vor. "Assign variables to child template in include tag Django." *Html - Assign variables to child template in include tag Django - Stack Overflow*. N.p., 24 July 2012. Web. 22 May 2017. <[`http://stackoverflow.com/a/11639689`](http://stackoverflow.com/a/11639689)>.
