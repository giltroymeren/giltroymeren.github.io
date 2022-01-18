---
layout: post
title:  "Notes on Day 22 of 30 Days of JavaScript"
date:   2022-01-19
categories: notes series javascript javascript30
---

## Table of Contents
* TOC
{:toc}

## Lessons Learned

### `Element.getBoundingClientRect()`

This method *returns a `DOMRect` object providing information about the size of an element and its position relative to the viewport*.

~~~ javascript
const containerCoords = document.querySelector('.container').getBoundingClientRect();
~~~

A `DOMRect` object has the following structure:

~~~ javascript
DOMRect
  bottom: 289
  height: 221.953125
  left: 154
  right: 504
  top: 67.046875
  width: 350
  x: 154
  y: 67.046875
  [[Prototype]]: DOMRect
~~~

This method is ideal for features that need a certain element's position within the viewable browser and perform specific operations on it. 

The project is one example: finding the cursor-pointed anchor tag within the page and making sure the lone `span` element to follow each of these accurately in the page as their pseudo-background.

## References
* "Element.getBoundingClientRect() - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 23 Sep. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/Element/getBoundingClientRect`](https://developer.mozilla.org/en-US/docs/Web/API/Element/getBoundingClientRect)>.
* "Follow Along Links" *wesbos.com*, uploaded by Wes Bos, 8 Dec. 2016, <[`javascript30.com`](https://javascript30.com/)>.
