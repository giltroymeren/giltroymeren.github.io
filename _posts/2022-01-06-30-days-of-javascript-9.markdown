---
layout: post
title:  "Notes on Day 9 of 30 Days of JavaScript"
date:   2022-01-06
categories: notes series javascript javascript30
---

The console dev tools in any browser is a developer's bestfriend for doing quick DOM changes and debugging.

## Table of Contents
* TOC
{:toc}

## Lessons Learned

### `console.clear()`

This clears the console's contents. In Chrome it will show the message "*Console was cleared*" after.

### `console.dir()`

It displays an interactive list of the properties of the specified JavaScript object.

~~~ javascript
<div id="container">Hello World</div>

const container = document.getElementById('container')
console.dir(container)

[object HTMLDivElement] {
  accessKey: "",
  addEventListener: function addEventListener() { [native code] },
  after: function after() { [native code] },
  align: "",
  animate: function animate() { [native code] },
  append: function append() { [native code] },
  appendChild: function appendChild() { [native code] },
  ...
  tagName: "DIV",
  TEXT_NODE: 3,
  textContent: "Hello World",
  title: "",
  ...
}
~~~

### `console.count()`

It counts the number of calls to the `String` parameter (can be of `Number` type) provided.

~~~ javascript
for(let i = 0; i < 5; i++) {
  if(i % 2 === 0) console.count('even')
  else console.count('odd')
}

> even: 1
> odd: 1
> even: 2
> odd: 2
> even: 3
~~~

A console `clear()` will remove all data accummulated, effectively resetting all previous declarations.

~~~ javascript
console.count('even')
> even: 1

console.count('odd')
> odd: 1
~~~

### `console.countReset()`

This removes the instance of the `String` parameter from the declared `count()` counter.

~~~ javascript
for(let i = 0; i < 5; i++) {
  if(i % 2 === 0) console.count('even')
  else console.count('odd')
}

console.countReset('even')
console.count('even')
> even: 1

console.countReset('odd')
console.count('odd')
> odd: 1
~~~

If the provided parameter is not found in the counter, it will show the following warning message:

~~~
Count for '' does not exist
~~~

### `console.time()`

This method starts a timer you can use to track how long an operation takes with a label.
`timeEnd()` is used to place where in the code the timer should stop and it will log the total time.

~~~ javascript
const timerLabel = 'get colors'
console.time(timerLabel)
fetch('https://api.sampleapis.com/csscolornames/colors')
  .then(response => response.json())
  .then(data => {
    console.timeEnd(timerLabel)
    console.log(data)
  });

> get colors: 1018.40185546875 ms
> Array(954) [
>   0: {id: 1, name: 'purple', hex: '#7e1e9c'}
>   1: {id: 2, name: 'green', hex: '#15b01a'}
>   2: {id: 3, name: 'blue', hex: '#0343df'}
>   ...
> ]
~~~

## References
* "14 Must Know Dev Tools Tricks" *wesbos.com*, uploaded by Wes Bos, 8 Dec. 2016, <[`javascript30.com`](https://javascript30.com/)>.
* "Console.count() - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 14 Sept. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/console/count`](https://developer.mozilla.org/en-US/docs/Web/API/console/count)>.
* "Console.dir() - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 15 Sept. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/console/dir`](https://developer.mozilla.org/en-US/docs/Web/API/console/dir)>.
* "Console.time() - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 15 Sept. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/console/time`](https://developer.mozilla.org/en-US/docs/Web/API/console/time)>.
