---
layout: post
title:  "Notes on Day 14 of 30 Days of JavaScript"
date:   2022-01-11
categories: notes series javascript javascript30
---

## Table of Contents
* TOC
{:toc}

## Lessons Learned

### `Array.prototype.concat()`

*The `concat()` method is used to merge two or more arrays. This method does not change the existing arrays, but instead returns a new array.*

~~~ javascript
const start = [1, 2, 3, 4, 5]
const end = [6, 7, 8, 9, 10]

start.concat(end)
> [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

console.log(start)
> [1, 2, 3, 4, 5]

start.concat()
> [1, 2, 3, 4, 5]
~~~

### `Object.assign()`

This method *copies all enumerable own properties from one or more source objects to a target object. It returns the modified target object.*

It has the following syntax:

~~~ javascript
Object.assign(<destination object>, <source object>, ...args)
~~~

Some examples:

~~~ javascript
const square = { sides: 4, color: 'blue' }
const rectangle = Object.assign({}, square, { perimeter: '2L + 2W' })

console.log(rectangle)
> {sides: 4, color: 'blue', perimeter: '2L + 2W'}

console.log(square)
> {sides: 4, color: 'blue'}
~~~

For *deep copy*, it is not recommended especially in React-Redux operations that need to handle complex states.

## References
* "Array.prototype.concat() - Web APIs \| MDN." *MDN Web Docs*, Mozilla Web Network, 17 Sep. 2021, <[`developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign)>.
* "Object.assign() - Web APIs \| MDN." *MDN Web Docs*, Mozilla Web Network, 17 Nov. 2021, <[`developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/concat`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/concat)>.
* "Objects and Arrays - Reference VS Copy" *wesbos.com*, uploaded by Wes Bos, 8 Dec. 2016, <[`javascript30.com`](https://javascript30.com/)>.
