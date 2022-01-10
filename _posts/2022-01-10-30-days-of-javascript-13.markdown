---
layout: post
title:  "Notes on Day 13 of 30 Days of JavaScript"
date:   2022-01-10
categories: notes series javascript javascript30
---

## Table of Contents
* TOC
{:toc}

## Lessons Learned

### debounce

The concept of debounce in JavaScript refers to limiting events, usually trigerred by users, to at least once per a given stimulus (say clicks, scroll or typing on a text field).

The following is a simple implementation:

~~~ javascript
function debounce(func, timeout = 300){
  let timer;
  return (...args) => {
    clearTimeout(timer);
    timer = setTimeout(() => { func.apply(this, args); }, timeout);
  };
}
~~~

This method wraps a callback `func` into a timeout and executes it given a time period. It also makes sure that only one `timer` object exists at a time.

### `Window.scrollX` and `Window.scrollY`

*The read-only `scrollX` property of the `Window` interface returns the number of pixels that the document is currently scrolled horizontally. This value is subpixel precise in modern browsers, meaning that it isn't necessarily a whole number. You can get the number of pixels the document is scrolled vertically from the `scrollY` property.*

~~~ javascript
const container = document.getElementById('object')
let xPosition = window.scrollX
let yPosition = window.scrollY

if(xPosition > 100 && yPosition < 50) {
  ...
}
~~~

### `HTMLElement.offsetTop`

*The `HTMLElement.offsetTop` read-only property returns the distance of the outer border of the current element relative to the inner border of the top of the `offsetParent` node*.

~~~ javascript
const container = document.getElementById('object')

if(container.offsetTop > 100) {
  ...
}
~~~

## References
* "HTMLElement.offsetTop - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 15 Sep. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/HTMLElement/offsetTop`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/offsetTop)>.
* Polesny, Ondrej. "Debounce – How to Delay a Function in JavaScript (JS ES6 Example)." *freecodecamp.org*, 8 Mar. 2021, <[`www.freecodecamp.org/news/javascript-debounce-example`](freecodecamp.org/news/javascript-debounce-example)>.
* "Slide In on Scroll" *wesbos.com*, uploaded by Wes Bos, 8 Dec. 2016, <[`javascript30.com`](https://javascript30.com/)>.
* "Window.scrollX - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 15 Sep. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/Window/scrollX`](https://developer.mozilla.org/en-US/docs/Web/API/Window/scrollX)>.
* "Window.scrollY - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 15 Sep. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/Window/scrollY`](https://developer.mozilla.org/en-US/docs/Web/API/Window/scrollY)>.