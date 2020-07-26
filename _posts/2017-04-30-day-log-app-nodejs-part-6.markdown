---
layout: post
title:  "Day Log App in Node.js Part 6: Delete a DayLog"
date:   2017-04-30
categories: projects notes node.js express series daylogapp
---

This post will discuss how to implement the deletion a DayLog.

## Table of Contents
* TOC
{:toc}

## Development

Each `DayLog` is removable and this should be accessible from the home page with the "Delete" button in the table.

![DayLog index page - not empty table](/assets/images/posts/2017-04-27-day-log-app-nodejs-part-2/daylog-table.png){:class="img-responsive"}

> **NOTE**: I decided to exclude the BDD testing in this part because I am having problems in implementing the edit test cases.

### Update the link

In `/views/daylog/index.ejs`. update the `action` path of the "Delete" button `form` from:

~~~ html
<form action="/<%= daylog.slug %>/delete" method="POST">
~~~

to the pattern `/daylog/:slug/delete`:

~~~ html
<form action="/daylog/<%= daylog.slug %>/delete" method="POST">
~~~

In addtion the method override for `DELETE` is already in the form.

~~~ html
<input type='hidden' value='DELETE' name='_method'>
~~~

### Add the route

> **NOTE**: Make sure that there's an exisiting `DayLog` in the database.

If we click the "Delete" button right now it should return a `404` so this route should be added:

~~~ javascript
router.delete('/:slug/delete', DayLogController.delete);
~~~

This should recognize the pattern `/daylog/:slug/delete` as intended.

### Delete the `DayLog`

Add the method `delete`:

~~~ javascript
delete: (req, res) => {
    DayLogModel.findOne({ "slug": req.url.split("/")[1] },
        (err, daylog) => {
            if(err) return console.error(err);

            daylog.remove((err, daylog) => {
                if(err) return console.error(err);

                console.info('Deleting: "' + daylog.title + '"');

                res.format({
                    html: () => { res.redirect("/daylog"); }
                });
            });
        }
    );
}
~~~

Re-run the application and deleting an entry in the table should remove it afterwards.

## References
* Coleman, Kendrick. "How to Create a Complete Express.js Node.js MongoDB CRUD and REST Skeleton." *Airpair*. N.p., 2015. Web. 20 Apr. 2017. <[`https://www.airpair.com/javascript/complete-expressjs-nodejs-mongodb-crud-skeleton`](https://www.airpair.com/javascript/complete-expressjs-nodejs-mongodb-crud-skeleton)>.
* Mangela, Siddhesh. "Siddacool/airpair-crud-app." *GitHub*. N.p., 23 Mar. 2016. Web. 20 Apr. 2017. <[`https://github.com/siddacool/airpair-crud-app`](https://github.com/siddacool/airpair-crud-app)>.
