---
layout: post
title:  "Notes on Day 10 of 30 Days of JavaScript"
date:   2022-01-07
categories: notes series javascript javascript30
---

The console dev tools in any browser is a developer's bestfriend for doing quick DOM changes and debugging.

## Table of Contents
* TOC
{:toc}

## Lessons Learned

~~~ javascript
<input />
<p></p>

const input = document.querySelector('input')
~~~

### `KeyboardEvent.shiftKey`

A boolean value that indicates if the <kdb>Shift</kbd> key was pressed (`true`) or not (`false`) when the event occurred.

~~~ javascript
input.addEventListener('keydown', (e) => console.log(e.shiftKey))
~~~

### `MouseEvent.shiftKey`

A boolean value that indicates whether the <kdb>Shift</kbd> key was pressed (`true`) or not (`false`) together with mouse events.

The code below logs `true` if we click the `input` while pressing down on the <kdb>Shift</kbd> key simultaneously.

~~~ javascript
input.addEventListener('mousedown', (e) => console.log(e.shiftKey))
~~~

### `GlobalEventHandlers.onkeydown`

This fires when the user presses a keyboard key. `onkeypress` and `keypress` are deprecated versions of this.

~~~ javascript
const logs = document.querySelector('p')

input.addEventListener('keydown', (e) => logs.textContent = e.code)
~~~

## References
* "GlobalEventHandlers.onkeydown - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 5 Nov. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/GlobalEventHandlers/onkeydown`](https://developer.mozilla.org/en-US/docs/Web/API/GlobalEventHandlers/onkeydown)>.
* "Hold Shift to Check Multiple Checkboxes" *wesbos.com*, uploaded by Wes Bos, 8 Dec. 2016, <[`javascript30.com`](https://javascript30.com/)>.
* "KeyboardEvent.shiftKey - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 15 Sept. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/shiftKey`](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/shiftKey)>.
* "MouseEvent.shiftKey - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 15 Sept. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/MouseEvent/shiftKey`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/shiftKey)>.
