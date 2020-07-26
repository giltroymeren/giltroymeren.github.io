---
layout: post
title:  "Day Log App in Django Part 7: Edit a DayLog"
date:   2017-05-24
categories: projects notes python django series daylogapp
---

This post will discuss how to implement the editing of an existing `Daylog` through a form.

## Table of Contents
* TOC
{:toc}

## Steps

### Show the form

#### Create test

The test will simply create a `Daylog` instance and expect the response from the server to contain the expected form attributes.

~~~ python
def test_edit_view(self):
    daylog = create_daylog()
    daylog.save()

    self.assertTrue(Daylog.objects.filter(slug = daylog.slug).count())

    response = self.client.get(reverse('daylog:edit', args=(daylog.slug,)))

    self.assertEqual(response.context['action'], 'daylog:edit')
    self.assertEqual(response.context['operation'], 'edit')
~~~

#### Set URL

Add the following to `api/urls.py`:

~~~ python
url(r'^(?P<slug>.+)/edit$', views.edit, name='edit'),
~~~

#### Create `edit` method

Same as the implementation of [`details`]({{ site.baseurl }}{% post_url 2017-05-23-day-log-app-django-part-5 %}) and [`delete`]({{ site.baseurl }}{% post_url 2017-05-23-day-log-app-django-part-6 %}), the `slug` from the URL is validated before the form is shown.

~~~ python
def edit(request, slug):
    try:
        daylog = Daylog.objects.get(slug = slug)
    except Daylog.DoesNotExist:
        messages.error(request, "Day Log does not exist.", "danger")
        return redirect('daylog:get_list')

    return render(request, 'daylog/form.html', {
        'daylog': daylog,
        'action': 'daylog:edit',
        'operation': 'edit',
    })
~~~

#### Update edit link

Changed the edit link in `daylog/templates/daylog/index.html` from:

~~~ html
/daylog/{{ daylog.slug }}/edit
~~~

to

~~~ html
{% raw %}
{% url 'daylog:edit' daylog.slug %}
{% endraw %}
~~~

![Edit Daylog page](/assets/images/posts/2017-05-23-day-log-app-django-part-7/edit.png){:class="img-responsive"}

The test `test_edit_view` should pass.

### Submit the form

#### Create test

~~~ python
from .forms import DaylogForm

...

def test_edit_existing_daylog_without_changes(self):
    old_daylog = create_daylog()
    old_daylog.save()

    self.assertTrue(Daylog.objects.filter(slug = old_daylog.slug).count())

    new_daylog = old_daylog.__dict__

    response = self.client.post(reverse('daylog:edit', args=(old_daylog.slug,)),
        new_daylog, follow=True)

    for message in response.context['messages']:
        self.assertEqual(message.message, "Day Log edited.")
        self.assertEqual(message.extra_tags, "info")
        break;

def test_edit_existing_daylog_with_valid_changes(self):
    old_daylog = create_daylog()
    old_daylog.save()

    self.assertTrue(Daylog.objects.filter(slug = old_daylog.slug).count())

    new_daylog = old_daylog.__dict__
    new_daylog['title'] = "New title"

    response = self.client.post(reverse('daylog:edit', args=(old_daylog.slug,)),
        new_daylog, follow=True)

    for message in response.context['messages']:
        self.assertEqual(message.message, "Day Log edited.")
        self.assertEqual(message.extra_tags, "info")
        break;
~~~

There is no need to check for existing `slug` and `log_at` values because they are disabled for modification during edit.

#### Update `edit` method

There are a few key changes in the back-end to make the edit submission work.

First, `DaylogForm` has a new parameter `operation` which will help in the validation to skip `slug` and `log_at` because they already exist in the database.

Next, is the submitted form passes validation then its values are saved together with the key attributes of the original `Daylog` object:

* `id`
* `created_at`
* `modified_at`

~~~ python
def edit(request, slug):
    try:
        old_daylog = Daylog.objects.get(slug = slug)

        if request.method == 'POST':
            daylog_form = DaylogForm(request.POST, operation='edit')
            if daylog_form.is_valid():
                new_daylog = daylog_form.instance
                new_daylog.id = old_daylog.id
                new_daylog.created_at = old_daylog.created_at
                new_daylog.modified_at = old_daylog.modified_at
                new_daylog.save()
                messages.info(request, "Day Log edited.", "info")
                return redirect('daylog:get_list')
            else:
                return render(request, 'daylog/form.html', {
                    'daylog': request.POST,
                    'action': 'daylog:edit',
                    'operation': 'edit',
                    'errors': daylog_form.errors
                 })
    except Daylog.DoesNotExist:
        messages.error(request, "Day Log does not exist.", "danger")
        return redirect('daylog:get_list')

    return render(request, 'daylog/form.html', {
        'daylog': old_daylog,
        'action': 'daylog:edit',
        'operation': 'edit',
    })
~~~

Add these new methods to the `DaylogForm` class in `daylog/forms.py`:

~~~ python
def __init__(self, *args, **kwargs):
    self.operation = kwargs.pop('operation', None)
    super(DaylogForm, self).__init__(*args, **kwargs)

def full_clean(self):
    super(DaylogForm, self).full_clean()

    if self.operation == 'edit':
        if 'slug' in self._errors:
            del self._errors['slug']
        if 'log_at' in self._errors:
            del self._errors['log_at']

    return self.cleaned_data
~~~

`full_clean` is the overriden method where the error messages for the two attributes are just removed during edit.

#### Update edit form

Go to `templates/daylog/index.html` and change the edit link from:

~~~ python
/daylog/{{ daylog.slug }}/edit
~~~

to

~~~ python
{% raw %}
{% url 'daylog:edit' daylog.slug %}
{% endraw %}
~~~

Additionally update the `readonly` conditional in `templates/daylog/form.html` of "Date of log" to:

~~~ python
{% raw %}
{% if operation == 'view' or operation == 'edit' %}readonly{% endif %}
{% endraw %}
~~~

## References
* "Documentation." *Writing your first Django app, part 3 &#124; Django documentation &#124; Django*. N.p., n.d. Web. 19 May 2017. <[`https://docs.djangoproject.com/en/1.11/intro/tutorial03/`](https://docs.djangoproject.com/en/1.11/intro/tutorial03/)>.
* "Documentation." *Writing your first Django app, part 4 &#124; Django documentation &#124; Django*. N.p., n.d. Web. 19 May 2017. <[`https://docs.djangoproject.com/en/1.11/intro/tutorial04/`](https://docs.djangoproject.com/en/1.11/intro/tutorial04/)>.
* Chrisv. "Django unit testing for form edit." *Stack Overflow*. N.p., 8 Oct. 2012. Web. 21 May 2017. <[`https://stackoverflow.com/a/12782253`](https://stackoverflow.com/a/12782253)>.
* Cushman, Jack. "Override data validation on one django form element." *Stack Overflow*. N.p., 25 Mar. 2013. Web. 20 May 2017. <[`http://stackoverflow.com/a/16745735`](http://stackoverflow.com/a/16745735)>.
* Freitas, Vitor. "How to Implement CRUD Using Ajax and Json." *Simple is Better Than Complex*. Simple is Better Than Complex, 15 Nov. 2016. Web. 19 May 2017. <[`https://simpleisbetterthancomplex.com/tutorial/2016/11/15/how-to-implement-a-crud-using-ajax-and-json.html`](https://simpleisbetterthancomplex.com/tutorial/2016/11/15/how-to-implement-a-crud-using-ajax-and-json.html)>.
* Freylis. "How can I turn Django Model objects into a dictionary and still have their foreign keys?" *Python - How can I turn Django Model objects into a dictionary and still have their foreign keys? - Stack Overflow*. N.p., 12 Sept. 2012. Web. 23 May 2017. <[`https://stackoverflow.com/a/12383051`](https://stackoverflow.com/a/12383051)>.
* Gikera, Jee. "Build a REST API with Django – A Test Driven Approach: Part 1." *Scotch*. N.p., 6 Feb. 2017. Web. 19 May 2017. <[`https://scotch.io/tutorials/build-a-rest-api-with-django-a-test-driven-approach-part-1`](https://scotch.io/tutorials/build-a-rest-api-with-django-a-test-driven-approach-part-1)>.
* Laffuste. "Django: Get Model instance from Form without saving." *Stack Overflow*. N.p., 9 May 2016. Web. 21 May 2017. <[`https://stackoverflow.com/a/37110602`](https://stackoverflow.com/a/37110602)>.
* Nikita, Sobolev. "Django: save() vs update() to update the database?" *Stack Overflow*. N.p., 26 May 2015. Web. 21 May 2017. <[`https://stackoverflow.com/a/30453181`](https://stackoverflow.com/a/30453181)>.
* Roseman, Daniel. "How to pass data to clean method in Django." *Stack Overflow*. N.p., 2 Oct. 2013. Web. 22 May 2017. <[`http://stackoverflow.com/a/19144599`](http://stackoverflow.com/a/19144599)>.





