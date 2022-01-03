---
layout: post
title:  "Notes on Day 6 of 30 Days of JavaScript"
date:   2022-01-03
categories: notes series javascript javascript30
---

There are many ways to access data from APIs and `fetch` is a common starting point in learning dynamic data.
I remember my Internet Programming class in 2012 and this was our introduction to the world of AJAX.

## Table of Contents
* TOC
{:toc}

## Lessons Learned

### `fetch()`

This method "starts the process of fetching a resource from the network, returning a promise which is fulfilled once the response is available" and it returns a `Response` object containing the data from the source point. 

Note that it has no support in Internet Explorer browsers.

### `ReadableStream`

It representes a readable stream of data and it is the type of the `body` attribute of a `fetch` `Response` object.

~~~ javascript
fetch(url).then(response => console.log(response))

Response {
  body: ReadableStream {
    locked: false,
    [[Prototype]]: ReadableStream
  },
  bodyUsed: false,
  headers: Headers {},
  ok: true,
  redirected: false,
  ...
}
~~~

Note that it has no support in Internet Explorer browsers.

## References
* "Ajax Type Ahead." *wesbos.com*, uploaded by Wes Bos, 8 Dec. 2016, <[`javascript30.com`](https://javascript30.com/)>.
* "fetch() - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 30 Oct. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/fetch`](https://developer.mozilla.org/en-US/docs/Web/API/fetch)>.
* "ReadableStream - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 14 Sept. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/ReadableStream`](https://developer.mozilla.org/en-US/docs/Web/API/ReadableStream)>.