---
layout: post
title:  "Day Log App in Node.js Part 4: Show a DayLog"
date:   2017-04-29
categories: projects notes node.js express zombie.js series daylogapp
---

This post will discuss the view page for a `DayLog`.

It has been implemented already in the [create `DayLog` post]({{ site.baseurl }}{% post_url 2017-04-29-day-log-app-nodejs-part-3 %}) with a test based from that feature's flow. However the test for accessing by clicking the `DayLog`'s title in the home page has not been done yet.

## Table of Contents
* TOC
{:toc}

## Development

### Make the failing test

The test is another `describe` block after `User visits /daylog/create`.

~~~ javascript
describe('Clicks "' + DAYLOG.title + '"', () => {
    it('should show the view form with the Day Log\'s values in readonly or disabled fields', () => {
        DayLogModel.create(DAYLOG);
        browser.visit(BASE_URL, () => {
            browser.clickLink('a[href="/daylog/' + DAYLOG.slug + '"]', () => {
                browser.assert.success();
                browser.assert.url(BASE_URL + '/' + DAYLOG.slug);
                expect(browser.assert.text('.card-header', 'View Day Log')).to.be.true;

                assertFormHasCreatedDayLogValues();

                browser.assert.element('textarea[name="title"][readonly]');
                browser.assert.element('input[name="slug"][readonly]');
                browser.assert.element('input[name="location"][readonly]');
                browser.assert.element('input[name="logAt"][readonly]');
                browser.assert.elements('input[name="category"][disabled]', 3);
            });
        });
    });

    afterEach((done) => {
        mongoose.connection.dropDatabase();
        done();
    });
});
~~~

It will click the title of the just-created `DayLog` in the table where the page should show the form with the details. Next requirement is that all fields must be inaccessible to edit by the user.

I also made a new method to check the fields if they have the same values as the `DayLog` constant used for creation:

~~~ javascript
function assertFormHasCreatedDayLogValues() {
    browser.assert.text('textarea[name="title"]', DAYLOG.title);
    expect(browser.field('input[name="slug"]', DAYLOG.slug)).to.be.true;
    expect(browser.field('input[name="location"]', DAYLOG.location)).to.be.true;
    expect(browser.field('input[name="logAt"]', DAYLOG.logAt)).to.be.true;
    browser.assert.element('input[name="category"][value="' + DAYLOG.category
        + '"]:checked');
}
~~~

Do not forget to update the original `it` block in `Click submit button with valid form`:

~~~ javascript
it('should show the view form with the submitted values', () => {
    assertFormHasCreatedDayLogValues();
});
~~~

### Run the tests

Since the view feature has been implemented already, the test should pass smoothly.

~~~ console
  User visits /daylog
    ✓ should show an alert when no DayLogs are available
    Insert one Day Log in database
      ✓ should show the Day Log in a table when it is added
    Clicks "Create Day Log" button
      ✓ should go to /daylog/create

  User visits /daylog/create
    Clicks submit button with empty form
      ✓ should show 5 "required" error messages near the 5 fields
    Clicks submit button with valid form
      ✓ should show the DayLog view page
      ✓ should show the view form with the submitted values
      ✓ should be in the DayLog home page table

  Clicks "Lorem ipsum dolor sit amet"
    ✓ should show the view form with the Day Log's values in readonly or disabled fields


  8 passing (11s)
~~~

## Problems

The original format of the test case was supposed to have separate `it` blocks after clicking the link to make the test descriptive:

~~~ javascript
describe('Clicks "' + DAYLOG.title + '"', () => {
    before((done) => {
        DayLogModel.create(DAYLOG);
        browser.visit(BASE_URL, done);
    });

    before((done) => {
        browser.clickLink('a[href="/daylog/' + DAYLOG.slug + '"]', done);
    });

    it('should show the View page', () => {
        browser.assert.success();
        browser.assert.url(BASE_URL + '/' + DAYLOG.slug);
        expect(browser.assert.text('.card-header', 'View Day Log')).to.be.true;
    });

    it('should show the view form with the Day Log\'s values', () => {
        assertFormHasCreatedDayLogValues();
    });

    it('should have all the fields to be readonly or disabled', () => {
        browser.assert.element('textarea[name="title"][readonly]');
        browser.assert.element('input[name="slug"][readonly]');
        browser.assert.element('input[name="location"][readonly]');
        browser.assert.element('input[name="logAt"][readonly]');
        browser.assert.elements('input[name="category"][disabled]', 3);
    });

    afterEach((done) => {
        mongoose.connection.dropDatabase();
        done();
    });
});
~~~

However this setup fails because it claims that the created `DayLog` is not present in `/daylog` (second `before` block) with the following logs:

~~~ console
  Clicks "Lorem ipsum dolor sit amet"
    1) "before all" hook


  7 passing (13s)
  1 failing

  1) Clicks "Lorem ipsum dolor sit amet" "before all" hook:
     AssertionError: No link matching 'a[href="/daylog/lorem-ipsum"]'
      at Browser.clickLink (node_modules/zombie/lib/index.js:695:7)
      at Context.before (test/functional/daylog.test.js:134:17)
      at EventLoop.done (node_modules/zombie/lib/eventloop.js:589:11)
      at Immediate.<anonymous> (node_modules/zombie/lib/eventloop.js:688:71)
~~~

I seriously spent time just to make it pass, trying to avoid putting all assertions inside the `visit` or `clickLink` methods, and unfortunately I cannot find other way to accomplish what I initially wanted.
