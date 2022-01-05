---
layout: post
title:  "Notes on Day 8 of 30 Days of JavaScript"
date:   2022-01-05
categories: notes series javascript javascript30
---

I rarely use the following array methods in my work so it is great to refresh my memory on their capabilities.

## Table of Contents
* TOC
{:toc}

## Lessons Learned

### `HTMLCanvasElement.getContext()`

This returns a drawing context on the canvas, or `null` if the context identifier is not supported, or the canvas has already been set to a different context mode.

~~~ javascript
<canvas id="draw" width="100%" height="100%"></canvas>

const myCanvas = document.querySelector('canvas#draw')
const myContext = myCanvas.getContext('2d')
~~~

Its first parameter, the `contextType` can be `2d`, `webgl`, `webgl2`, or `bitmaprenderer`. These will dictate the way the context will be renderered.

### `CanvasRenderingContext2D.globalCompositeOperation`

This is a property of the Canvas 2D API sets the type of compositing operation to apply when drawing new shapes. It requires the `2d` `contextType` for declaring a context.

~~~ javascript
<canvas id="draw" width="100%" height="100%"></canvas>

const myCanvas = document.querySelector('canvas#draw')
const myContext = myCanvas.getContext('2d')

myContext.globalCompositeOperation = 'color-burn'
~~~

Its values are similar to Photoshop or other photo editing tool's filter and manipulation presets.

### `mouse` events

| Property    | Definition                                                                                                                                                 |
|-------------|------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `mouseup`   | This is fired at an element when a cursor is released while the pointer is located inside it.                                                              |
| `mousedown` | This is fired at an element when a cursor is pressed while the pointer is located inside it.                                                               |
| `mousemove` | This is fired at an element when a pointing device is moved while the cursor's hotspot is inside it.                                                       |
| `mouseout`  | This is fired at an element when a pointing device is used to move the cursor so that it is no longer contained within the element or one of its children. |

## References
* "CanvasRenderingContext2D.globalCompositeOperation - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 14 Sept. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/globalCompositeOperation`](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/globalCompositeOperation)>.
* "Element: mousedown event - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 15 Sep. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/Element/mousedown_event`](https://developer.mozilla.org/en-US/docs/Web/API/Element/mousedown_event)>.
* "Element: mousemove event - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 15 Sep. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/Element/mousemove_event`](https://developer.mozilla.org/en-US/docs/Web/API/Element/mousemove_event)>.
* "Element: mouseout event - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 15 Sep. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/Element/mouseout_event`](https://developer.mozilla.org/en-US/docs/Web/API/Element/mouseout_event)>.
* "Element: mouseup event - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 15 Sep. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/Element/mouseup_event`](https://developer.mozilla.org/en-US/docs/Web/API/Element/mouseup_event)>.
* "Fun with HTML5 Canvas" *wesbos.com*, uploaded by Wes Bos, 8 Dec. 2016, <[`javascript30.com`](https://javascript30.com/)>.
* "HTMLCanvasElement.getContext() - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 7 Dec. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/HTMLCanvasElement/getContext`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLCanvasElement/getContext)>.