---
layout: post
title:  "Day Log App in Django Part 5: Show a DayLog"
date:   2017-05-23
categories: projects notes python django series daylogapp
---
This post will discuss how to show a `Daylog`'s details in a page.

## Table of Contents
* TOC
{:toc}

## Steps

### Create test

The tests will check for a valid `slug` parameter.

~~~ python
def test_detail_view_with_a_valid_daylog(self):
    daylog = create_daylog()
    response = self.client.get(reverse('daylog:details', args=(daylog.slug,)))

    self.assertEqual(response.context['daylog'], daylog)
    self.assertEqual(response.context['operation'], 'view')

def test_detail_view_with_an_invalid_daylog(self):
    response = self.client.get(reverse('daylog:details', args=('fake-slug',)))

    for message in response.context['messages']:
        self.assertEqual(message.message, "Day Log does not exist.")
        self.assertEqual(message.extra_tags, "danger")
        break;
~~~

### Set URL

Add the URL to view `Daylog`s by their `slug` attribute in `api/urls.py`. This will be passed to the `views.details` method.

~~~ python
url(r'^(?P<slug>.+)/$', views.details, name='details'),
~~~

### Create `details` method

Add the following method in `daylog/views.py`:

~~~ python
def details(request, slug):
    try:
        daylog = Daylog.objects.get(slug = slug)
    except Daylog.DoesNotExist:
        messages.error(request, "Day Log does not exist.", "danger")
        return get_list(request)

    return render(request, 'daylog/form.html', {
        'daylog': daylog,
        'operation': 'view'
    })
~~~

First the provided `slug` is fetched from the database. If it does not exist, then it should go back to the mhome page with an error message.

![Home page with Daylog DNE alert](/assets/images/posts/2017-05-23-day-log-app-django-part-5/daylog-dne-alert.png){:class="img-responsive"}

The file `daylog/form.html` is still used, only this time all fields are disabled and the form buttons not available.

![View Daylog page](/assets/images/posts/2017-05-23-day-log-app-django-part-5/view.png){:class="img-responsive"}

## References
* "Documentation." *Writing your first Django app, part 3 &#124; Django documentation &#124; Django*. N.p., n.d. Web. 19 May 2017. <[`https://docs.djangoproject.com/en/1.11/intro/tutorial03/`](https://docs.djangoproject.com/en/1.11/intro/tutorial03/)>.
* "Documentation." *Writing your first Django app, part 4 &#124; Django documentation &#124; Django*. N.p., n.d. Web. 19 May 2017. <[`https://docs.djangoproject.com/en/1.11/intro/tutorial04/`](https://docs.djangoproject.com/en/1.11/intro/tutorial04/)>.
* Freitas, Vitor. "How to Implement CRUD Using Ajax and Json." *Simple is Better Than Complex*. Simple is Better Than Complex, 15 Nov. 2016. Web. 19 May 2017. <[`https://simpleisbetterthancomplex.com/tutorial/2016/11/15/how-to-implement-a-crud-using-ajax-and-json.html`](https://simpleisbetterthancomplex.com/tutorial/2016/11/15/how-to-implement-a-crud-using-ajax-and-json.html)>.
* Gikera, Jee. "Build a REST API with Django – A Test Driven Approach: Part 1." *Scotch*. N.p., 6 Feb. 2017. Web. 19 May 2017. <[`https://scotch.io/tutorials/build-a-rest-api-with-django-a-test-driven-approach-part-1`](https://scotch.io/tutorials/build-a-rest-api-with-django-a-test-driven-approach-part-1)>.
* Mohsen. "Url pattern for alphanumeric(slug) string in django." *Python - Url pattern for alphanumeric(slug) string in django - Stack Overflow*. N.p., 4 July 2015. Web. 20 May 2017. <[`http://stackoverflow.com/a/31217155`](http://stackoverflow.com/a/31217155)>.