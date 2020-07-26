---
layout: post
title:  "Day Log App in Django Part 6: Delete a DayLog"
date:   2017-05-23
categories: projects notes python django series daylogapp
---

This post will discuss how to delete a `Daylog`.

## Table of Contents
* TOC
{:toc}

## Steps

### Create test

The tests will check for a valid `slug` parameter before deletion.

~~~ python
def test_delete_existing_daylog(self):
    daylog = create_daylog()
    daylog.save()

    self.assertTrue(Daylog.objects.filter(slug = daylog.slug).count())

    response = self.client.post(reverse('daylog:delete', args=(daylog.slug,)), follow=True)

    self.assertFalse(Daylog.objects.filter(slug = daylog.slug).count())

    for message in response.context['messages']:
        self.assertEqual(message.message, "Day Log deleted.")
        self.assertEqual(message.extra_tags, "info")
        break;

def test_delete_nonexistent_daylog(self):
    nonexistent_slug = "non-existent-slug"

    self.assertFalse(Daylog.objects.filter(slug = nonexistent_slug).count())

    response = self.client.post(reverse('daylog:delete', args=(nonexistent_slug,)), follow=True)

    for message in response.context['messages']:
        self.assertEqual(message.message, "Day Log does not exist.")
        self.assertEqual(message.extra_tags, "danger")
        break;
~~~

### Set URL

The URL for delete will be the `slug` followed by the word "delete":

~~~ python
url(r'^(?P<slug>.+)/delete$', views.delete, name='delete'),
~~~

### Create `delete` method

The `delete` method will use the `redirect` method after a successful deletion in order to prevent repetition in case the back feature of browsers is used.

~~~ python
from django.shortcuts import redirect, render
...
def delete(request, slug):
    try:
        daylog = Daylog.objects.get(slug = slug)
    except Daylog.DoesNotExist:
        messages.error(request, "Day Log does not exist.", "danger")
        return redirect('daylog:get_list')

    daylog.delete()
    messages.error(request, "Day Log deleted.", "info")
    return redirect('daylog:get_list')
~~~

### Update delete form

Go to `templates/daylog/index.html` and change the delete form from:

~~~ python
<form action="/daylog/{{ daylog.slug }}/delete" method="POST">
    <input type='hidden' value='DELETE' name='_method'>
~~~

to

~~~ html
{% raw %}
<form action="{% url 'daylog:delete' daylog.slug %}" method="POST">
    {% csrf_token %}
{% endraw %}
~~~

## References
* "Documentation." *Writing your first Django app, part 3 &#124; Django documentation &#124; Django*. N.p., n.d. Web. 19 May 2017. <[`https://docs.djangoproject.com/en/1.11/intro/tutorial03/`](https://docs.djangoproject.com/en/1.11/intro/tutorial03/)>.
* "Documentation." *Writing your first Django app, part 4 &#124; Django documentation &#124; Django*. N.p., n.d. Web. 19 May 2017. <[`https://docs.djangoproject.com/en/1.11/intro/tutorial04/`](https://docs.djangoproject.com/en/1.11/intro/tutorial04/)>.
* Freitas, Vitor. "How to Implement CRUD Using Ajax and Json." *Simple is Better Than Complex*. Simple is Better Than Complex, 15 Nov. 2016. Web. 19 May 2017. <[`https://simpleisbetterthancomplex.com/tutorial/2016/11/15/how-to-implement-a-crud-using-ajax-and-json.html`](https://simpleisbetterthancomplex.com/tutorial/2016/11/15/how-to-implement-a-crud-using-ajax-and-json.html)>.
* Gikera, Jee. "Build a REST API with Django – A Test Driven Approach: Part 1." *Scotch*. N.p., 6 Feb. 2017. Web. 19 May 2017. <[`https://scotch.io/tutorials/build-a-rest-api-with-django-a-test-driven-approach-part-1`](https://scotch.io/tutorials/build-a-rest-api-with-django-a-test-driven-approach-part-1)>.
