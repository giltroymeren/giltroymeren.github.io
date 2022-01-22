---
layout: post
title:  "Notes on Day 26 of 30 Days of JavaScript"
date:   2022-01-23
categories: notes series javascript javascript30 
---

## Table of Contents
* TOC
{:toc}

## Lessons Learned

### `CSSStyleDeclaration.setProperty()`

This sets new key-value pairs for the `style` attribute of a DOM element.

~~~ javascript
const container = document.querySelector(".container")
containter.style.setProperty("background", "#CCCCCC")
~~~

This is another alternative to the camelcased CSS attribute shorthand if you are not sure of their representation as a `CSSStyleDeclaration.cssText` property inline name.

~~~ javascript
const container = document.querySelector(".container")
containter.style.background = "#CCCCCC"
~~~

## References
* "CSSStyleDeclaration.cssText - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 15 Sep. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/CSSStyleDeclaration/cssText`](https://developer.mozilla.org/en-US/docs/Web/API/CSSStyleDeclaration/cssText)>.
* "CSSStyleDeclaration.setProperty() - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 14 Jan. 2022, <[`developer.mozilla.org/en-US/docs/Web/API/CSSStyleDeclaration/setProperty`](https://developer.mozilla.org/en-US/docs/Web/API/CSSStyleDeclaration/setProperty)>.
* "Stripe Follow Along Dropdown" *wesbos.com*, uploaded by Wes Bos, 8 Dec. 2016, <[`javascript30.com`](https://javascript30.com/)>.
