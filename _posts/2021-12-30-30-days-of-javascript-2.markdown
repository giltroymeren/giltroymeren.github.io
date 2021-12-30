---
layout: post
title:  "Notes on Day 2 of 30 Days of JavaScript"
date:   2021-12-30
categories: notes series javascript css 30_days_of_js
---

Today's topic mostly had basic JS code and some time mathematics. What stood out for me are the CSS animations discussed.

## Table of Contents
* TOC
{:toc}

## Lessons Learned

### `transform`

It is a CSS property that is capable of manipulating a DOM element in terms of looks. 

~~~ css
transform: rotate(0deg) scale(1.5) skew(1deg) translate(-187px);
-webkit-transform: rotate(0deg) scale(1.5) skew(1deg) translate(-187px);
-moz-transform: rotate(0deg) scale(1.5) skew(1deg) translate(-187px);
-o-transform: rotate(0deg) scale(1.5) skew(1deg) translate(-187px);
-ms-transform: rotate(0deg) scale(1.5) skew(1deg) translate(-187px);
~~~

Some of the attributes it could perform are:

- `matrix`
- `matrix3d`
- `perspective`
- `rotate`
- `rotate3d`
- `rotateX`
- `rotateY`
- `rotateZ`
- `translate`
- `translate3d`
- `translateX`
- `translateY`
- `translateZ`
- `scale`
- `scale3d`
- `scaleX`
- `scaleY`
- `scaleZ`
- `skew`
- `skewX`
- `skewY`

and can be combined into a single declaration to perform more complex transformations.

### `transform-origin`

It sets the axis where an element should move based on the applied `transform` attributes.

## References
* "CSS + JS Clock." *wesbos.com*, uploaded by Wes Bos, 8 Dec. 2016, <[`javascript30.com`](https://javascript30.com/)>.
* "transform - CSS: Cascading Style Sheets \| MDN." *MDN Web Docs*, Mozilla Developer Network, 25 Oct. 2021, <[`developer.mozilla.org/en-US/docs/Web/CSS/transform`](https://developer.mozilla.org/en-US/docs/Web/CSS/transform)>.
* "transform-origin - CSS: Cascading Style Sheets \| MDN." *MDN Web Docs*, Mozilla Developer Network, 13 Aug. 2021, <[`developer.mozilla.org/en-US/docs/Web/CSS/transform-origin`](https://developer.mozilla.org/en-US/docs/Web/CSS/transform-origin)>.