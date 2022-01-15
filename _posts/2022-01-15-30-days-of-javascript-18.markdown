---
layout: post
title:  "Notes on Day 18 of 30 Days of JavaScript"
date:   2022-01-15
categories: notes series javascript javascript30
---

## Table of Contents
* TOC
{:toc}

## Lessons Learned

### `NodeList`

*`NodeList` objects are collections of nodes, usually returned by properties such as `Node.childNodes` and methods such as `document.querySelectorAll()`*.

These are not of the array type and can be worked with common array methods by transforming them first into one:

~~~ javascript
const menuItems = document.querySelectorAll('ul.menu-items li')
NodeList.prototype.isPrototypeOf(menuItems)
> true

[...menuItems]
> [li, li, li, ...]

Array.from(menuItems)
> [li, li, li, ...]
~~~

## References
* "NodeList - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 15 Sep. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/NodeList`](https://developer.mozilla.org/en-US/docs/Web/API/NodeList)>.
* "Tally String Times with Reduce" *wesbos.com*, uploaded by Wes Bos, 8 Dec. 2016, <[`javascript30.com`](https://javascript30.com/)>.
* vineethbc. "How to Detect HTMLCollection/NodeList in JavaScript?" *Stack Overflow*, 26 Apr. 2016, <[`stackoverflow.com/questions/7238177/how-to-detect-htmlcollection-nodelist-in-javascript/36857902#36857902`](https://stackoverflow.com/questions/7238177/how-to-detect-htmlcollection-nodelist-in-javascript)>.