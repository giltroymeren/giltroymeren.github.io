---
layout: post
title:  "Notes on Day 1 of 30 Days of JavaScript"
date:   2021-12-29
categories: notes series javascript javascript30
---

This is the first entry of [Wes Bos' 30 Days of JavaScript](https://javascript30.com/) coding challenge to brush up on my JavaScript skills and learn new things that I might not be aware of!

Each post will just be a list of information and pieces of code, technique that interested me during the lesson.

## Table of Contents
* TOC
{:toc}

## Lessons Learned

### `keyup` and `keydown`

According to MDN, `keyup` is fired when a key is **released** while `keydown` is when a key is **pressed**.

~~~ javascript
const printCode = (event) => console.log(event)

window.addEventListener('keyup', printCode)
window.addEventListener('keydown', printCode)
~~~

You can test this by listening to the keyboard. For `keyup`, if we press hard on a key, the `printCode` will only execute after we lift from the key. On the other hand, `keydown` immediately executes `printCode` once the key is pressed.

We can use these information on how to incorporate these in our projects. Personally I have only encountered the use of `keyup` for handling inputs on React projects where we need to listen for text fields that deal with AJAX calls.

### `KeyboardEvent` object

In relation to the two event listeners in the previous entry, this object is returned if we access an `event` object from, as the name suggests, keyboard events.

~~~~ javascript
KeyboardEvent {
    charCode: 0
    code: "KeyA"
    key: "a"
    keyCode: 65
    ...
    type: "keydown"
    which: 65
    [[Prototype]]: KeyboardEvent
}
~~~~

There are five particular attributes that I found interesting because I initally thought they referred to the same *thing*:

- `KeyboardEvent.code`
- `KeyboardEvent.charCode`
- `KeyboardEvent.key`
- `KeyboardEvent.keyCode`
- `KeyboardEvent.which`

<br />

| Property    | Description                                                                                                                    |
|-------------|--------------------------------------------------------------------------------------------------------------------------------|
| `code`      | code value of the key based from [this table](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/code/code_values) |
| `charCode`* | a Unicode reference number of the key                                                                                          |
| `key`       | key value of the key based from [this](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/key/Key_Values)          |
| `keyCode`*  | either the [decimal ASCII](https://datatracker.ietf.org/doc/html/rfc20) code of the key or 0 if unidentifiable                 |
| `which`*    | same as with `keycode`                                                                                                         |

> * a deprecated property; `key` is encouraged as replacement

Some examples:

| Property   | a      | A      | 1        | !        | =       |
| ---------- | ------ | ------ | -------- | -------- | ------- |
| `code`     | "KeyA" | "KeyA" | "Digit1" | "Digit1" | "Equal" |
| `charCode` | 0      | 0      | 0        | 0        | 0       |
| `key`      | "a"    | "A"    | "1"      | "!"      | "="     |
| `keyCode`  | 65     | 65     | 49       | 49       | 187     |
| `which`    | 65     | 65     | 49       | 49       | 187     |

As of December 2021, we should only rely on `code` and `key` for keyboard-related events.

### `HTMLMediaElement.currentTime`

This was used in the exercise to control the overall execution time of the audio files that correspond to the key presses.

It is applicable to `HTMLMediaElement`s such as `<audio>` and `<video>` for controlling their playback time (in seconds).

~~~~ javascript
const audio = document.querySelector('audio.my-song')
audio.currentTime = 0
audio.play()
~~~~

### `HTMLMediaElement.play()`

This is used to execute the audio or video element in the page.

### `HTMLElement: transitionend` event

This event was used to listen to the DOM element's CSS `transition` property for any changes.
Note that the entire keyword is in lowercase.

~~~ javascript
const menuItems = document.querySelectorAll('#menu .menu-items')
menuItems.forEach(item => {
    item.addEventListener('transitionend', () => {
        item.classList.remove('hidden')
    })
})
~~~

### `Array.from()`

I believe this was my first time encountering the use of this method because I usually do straight assignment of array methods' results (`map`, `filter` and so on).

This was used in the exercise to get an array of the `document.querySelectorAll()` results. Below are the difference between the results of using and not using `Array.from()` with the aforementioned DOM method.

~~~ javascript
document.querySelectorAll('span')
// NodeList(5) [span, span, span.cm-variable, span.cm-property, span.cm-variable]

Array.from(document.querySelectorAll('span'))
// (5) [span, span, span.cm-variable, span.cm-property, span.cm-variable] 
~~~

It returns a new array from an iterable object.

~~~ javascript
console.log(Array.from('Hello World'))
// ['H', 'e', 'l', 'l', 'o', ' ', 'W', 'o', 'r', 'l', 'd']
~~~

## References
* "Array.from() - JavaScript \| MDN." *MDN Web Docs*, Mozilla Developer Network, 10 Dec. 2021, <[`developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/from`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/from)>.
* "Document: Keydown Event - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 26 Oct. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/Document/keydown_event`](https://developer.mozilla.org/en-US/docs/Web/API/Document/keydown_event)>.
* "Document: Keyup Event - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 14 Sept. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/Document/keyup_event`](https://developer.mozilla.org/en-US/docs/Web/API/Document/keyup_event)>.
* "HTMLElement: Transitionend Event - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 14 Sept. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/HTMLElement/transitionend_event`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/transitionend_event)>.
* "JavaScript Drum Kit." *wesbos.com*, uploaded by Wes Bos, 8 Dec. 2016, <[`javascript30.com`](https://javascript30.com/)>.
* "KeyboardEvent - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 16 Nov. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent`](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent)>.
