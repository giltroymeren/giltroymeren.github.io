---
layout: post
title:  "Notes on Day 27 of 30 Days of JavaScript"
date:   2022-01-24
categories: notes series javascript javascript30 
---

I learned a lot about mouse-related event listeners!

## Table of Contents
* TOC
{:toc}

## Lessons Learned

### Mouse events

The following are available event listeners for mouse or any pounting devices:

#### Element: `mousedown` event

`mousedown` is fired when a cursor is *pressed* while inside the area of the target element.

~~~ javascript
element.addEventListener('mousedown', () => element.classList.add('active'))
~~~

#### Element: `mouseup` event

`mouseup`, the opposite of `mousedown`, is fired when the cursor is *released* inside the area of the target element.

~~~ javascript
element.addEventListener('mouseup', () => element.classList.remove('active'))
~~~

#### Element: `mouseenter` event

`mouseenter` is fired when a cursor is moved so that it is located inside the target element with the event listener.

#### Element: `mouseleave` event

`mouseleave` is fired when a cursor is *moved out* of the target element with the event listener.

#### Element: `mousemove` event

`mousemove1 is fired when the cursor *is moved while the cursor's hotspot is inside* the target element with the event listener.

### `cursor: grabbing`

This cursor represents the "grabbing" or "closed hand" key and is usually used for drag and drop features.

~~~ css
.grabbing {
  cursor: grabbing;
  cursor: -moz-grabbing;
  cursor: -webkit-grabbing;
}
~~~

## References
* "Click and Drag to Scroll" *wesbos.com*, uploaded by Wes Bos, 8 Dec. 2016, <[`javascript30.com`](https://javascript30.com/)>.
* "cursor - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 29 Nov. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/CSS/cursor`](https://developer.mozilla.org/en-US/docs/Web/API/CSS/cursor)>.
* "Element: mousedown event - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 15 Sep. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/Element/mousedown_event`](https://developer.mozilla.org/en-US/docs/Web/API/Element/mousedown_event)>.
* "Element: mouseenter event - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 15 Sep. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/Element/mouseenter_event`](https://developer.mozilla.org/en-US/docs/Web/API/Element/mouseenter_event)>.
* "Element: mouseleave event - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 15 Sep. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/Element/mouseleave_event`](https://developer.mozilla.org/en-US/docs/Web/API/Element/mouseleave_event)>.
* "Element: mousemove event - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 15 Sep. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/Element/mousemove_event`](https://developer.mozilla.org/en-US/docs/Web/API/Element/mousemove_event)>.
* "Element: mouseup event - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 15 Sep. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/Element/mouseup_event`](https://developer.mozilla.org/en-US/docs/Web/API/Element/mouseup_event)>.
* Steve, J. "CSS for grabbing cursors (drag & drop)." *Stack Overflow*, 17 Apr. 2011, <[`stackoverflow.com/a/18294634`](https://stackoverflow.com/a/18294634)>.