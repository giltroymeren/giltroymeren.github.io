---
layout: post
title:  "Notes on Day 24 of 30 Days of JavaScript"
date:   2022-01-21
categories: notes series javascript javascript30 css layout
---

## Table of Contents
* TOC
{:toc}

## Lessons Learned

### Setting fixed (horizontal) navigation menus

1. Add a `scroll` event listener to a parent container 

    ~~~ javascript
    window.addEventListener('scroll', () => doSomething());
    ~~~

2. Determine the distance of the menu from the top and use this condition to check if it needs to be in a fixed position.

    All `HTMLElements` have the `offsetTop` property that returns their distance from the top of the viewport. The `scrollY` property on the other hand returns the *current* distance of the scrollable window from the top. 

    These two determine if the menu needs to be set to fixed or not.

    ~~~ javascript
    if(window.scrollY >= navElement.offsetTop) {
      document.body.classList.add("nav-fixed");
    }
    ~~~

3. Set the fixed selector to the parent in case other element siblings of the fixed menu need some adjustments.

    In line with this reminder, when the menu was set to fixed, the scrollable body had a "jerky" upwards motion as the menu was moved from the normal blocks. The solution was to set a top padding for the parent based on the menu's height (`offsetHeight`) to counter the "empty" space it left after becoming fixed.

    ~~~ javascript
    if(window.scrollY >= navElement.offsetTop) {
      document.body.style.paddingTop = nav.offsetHeight + 'px';
      document.body.classList.add("nav-fixed");
    }
    ~~~

    The implication of this parent with the fixed selector makes a lot of possibilities in terms of animating and designing other moving parts of the page in relation to this change of position. 

4. Finally add an opposite case if the scroll window is still not within the target breakpoint: simply set the padding back to zero and remove the fixed class.


    ~~~ javascript
    else {
      document.body.style.paddingTop = n0;
      document.body.classList.remove("nav-fixed");
    }
    ~~~

The resuable code is the following:

~~~ javascript
const navElement = document.querySelector('nav#menu');

const setFixedNav = () => {
  if (window.scrollY >= navElement.offsetTop) {
    document.body.style.paddingTop = nav.offsetHeight + 'px';
    document.body.classList.add("nav-fixed");
  } else {
    document.body.style.paddingTop = 0;
    document.body.classList.remove("nav-fixed");
  }
}

window.addEventListener('scroll', setFixedNav);
~~~

## References
* "Sticky Nav" *wesbos.com*, uploaded by Wes Bos, 8 Dec. 2016, <[`javascript30.com`](https://javascript30.com/)>.
