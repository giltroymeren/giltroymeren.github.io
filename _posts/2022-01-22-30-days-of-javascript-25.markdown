---
layout: post
title:  "Notes on Day 25 of 30 Days of JavaScript"
date:   2022-01-22
categories: notes series javascript javascript30 
---

Today's lecture focused on event listeners in JavaScript.

## Table of Contents
* TOC
{:toc}

## Lessons Learned

For example, we have the following nested structures and these will have `click` event listeners.

~~~ html
<div class="parent">
  <div class="child-1">
    <div class="child-0">
    </div>
  </div>
</div>
~~~

~~~ javascript
const divs = document.querySelectorAll("div")

const logClass = event => console.log(event.target.classList.value)

divs.forEach(div => div.addEventListener('click', logClass))
~~~

In this current setup of the event listener, here's the logs when each `div` is clicked:

| Element clicked | Log                                 |
|-----------------|-------------------------------------|
| `child-0`       | "child-0"<br>"child-0"<br>"child-0" |
| `child-1`       | "child-1"<br>"child-1"              |
| `parent`        | "parent"                            |

This is strange, especially for the innermost children wherein their classnames were printed multiple times as their tier from the outermost parent. Why did this happen?

Remember that each `div` is wrapped by an immediate parent and they are not individual, separate elements. The event listeners took into account this structure that is why `child-0`'s parent `child-1` up to the parent `parent` and so on. If all three `div`s where separate entities in a row or column, then each click will be counted based on the number of clicks done.

Lastly, the results from the table is still the same if `logClass()` was in the traditional function declaration:

~~~ javascript
function logClass(event) {
  console.log(event.target.classList.value)
}
~~~

The key similarity with the two is the `event.target` object which triggers from the same target element.

### `this` and arrow functions

Remember that arrow functions are not bound to the `this` keyword so the following `logClass()` implementation should raise an error:

~~~ javascript
// event.target and this should be equivalent
const logClass = event => console.log(this.classList.value)

> Uncaught TypeError: Cannot read properties of undefined (reading 'value')
~~~

The `this` keyword should work with a traditional function declaration because it binds with such features.

~~~ javascript
function logClass(event) {
  console.log(this.classList.value)
}
~~~

**However** `this.classList.value` has a different output compared to `event.target.value`:

| Element clicked | Log                                |
|-----------------|------------------------------------|
| `child-0`       | "child-0"<br>"child-1"<br>"parent" |
| `child-1`       | "child-1"<br>"parent"              |
| `parent`        | "parent"                           |

The logs will still be of the same number as the tier of the element clicked but now, its immediate parent's class is shown instead. The difference is now through the event bubbling from the source target that rises upwards to its parents in the DOM tree.

### `Event.stopPropagation()`

This method prevents the event bubbling of event listeners.

If added to either implementations from the previous `logClass()` methods:

~~~ javascript
function logClass(event) {
  event.stopPropagation()
  console.log(this.classList.value)
}
~~~

This new method should only log the class of the `div` clicked.

### `EventTarget.addEventListener()` optional parameters

Recall that the `addEventListener()` method has the following syntaxes:

~~~ javascript
element.addEventListener(type, listener);
element.addEventListener(type, listener, options);
element.addEventListener(type, listener, useCapture);
~~~

We will focus on the second one with the third `options` parameter, which is an object of available options *hat specifies characteristics about the event listener*.

In the context of our example `div` click listener before:

~~~ javascript
divs.forEach(div => div.addEventListener(
  'click', 
  logClass,
  {
    capture: false,
    once: false,
    ...
  }))

~~~

#### `EventTarget.addEventListener()` option: `capture`

Events are captured *upwards* (hence bubbling) so the output of our `logClass()` was from `child-0` to `parent`. 

`capture` takes a boolean, `false` by default and dictates where the capturing in upwards or downwards. 

If switched to true, then the method should print `parent` first then `child-0` last, downwards the DOM tree.

#### `EventTarget.addEventListener()` option: `once`

The `once` option is pretty straightforward - it determines if the method set to the listener will be fired *once* or multiple times (default).

Another feature if set to `true` is that the listener will be removed after firing once. So this means setting this also calls `removeEventListener()` to the element:

~~~ javascript
divs.forEach(div => div.addEventListener('click', logClass, { once: false }))

divs.forEach(div => div.removeEventListener('click'))
~~~

This is useful when we want some elements or events to just happen once a user or another process interacts with the trigger.

## References
* "Event Capture, Propagation, Bubbling and Once" *wesbos.com*, uploaded by Wes Bos, 8 Dec. 2016, <[`javascript30.com`](https://javascript30.com/)>.
* "Event.stopPropagation() - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 22 Oct. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/Event/stopPropagation`](https://developer.mozilla.org/en-US/docs/Web/API/Event/stopPropagation)>.
* "EventTarget.addEventListener() - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 7 Jan. 2022, <[`developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener`](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener)>.

