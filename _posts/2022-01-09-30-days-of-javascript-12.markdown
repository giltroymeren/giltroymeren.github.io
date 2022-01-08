---
layout: post
title:  "Notes on Day 12 of 30 Days of JavaScript"
date:   2022-01-09
categories: notes series javascript javascript30
---

As a millenial this is my firt time to learn about the "KONAMI Code" from 1980s vide0 game culture.

According to Edwards oh howtogeek.com:

> Up, Up, Down, Down, Left, Right, Left, Right, B, A. It’s called the Konami Code, and it often meant the difference between life and death in a video game back in the 1980s.

## Table of Contents
* TOC
{:toc}

## Lessons Learned

### `Array.prototype.push()`

`push` adds one or more elements to the *end* of an array and returns the length of the array.

~~~ javascript
const arr = ['a', 'b', 'c', 'd', 'e']
arr.push('f')
> 6

arr.push('g', 'h', 'i', 'j')
> 10

arr.push([1, 2, 3])
> 11

console.log(arr)
> ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', [1, 2, 3]]
~~~

### `Array.prototype.splice()`

*The `splice()` method changes the contents of an array by removing or replacing existing elements and/or adding new elements in place*

~~~ javascript
const arr = ['a', 'b', 'c', 'd', 'e']
arr.splice(0, arr.length-1)
> ['a', 'b', 'c', 'd']

arr.splice(0, 1)
> ['a']

arr.splice(0, 1, 'A')
console.log(arr)
> ['A', 'b', 'c', 'd', 'e']

arr.splice(0, arr.length, 'f')
console.log(arr)
> ['f']
~~~

### `Array.prototype.join()`

*The `join()` method creates and returns a new string by concatenating all of the elements in an array (or an array-like object), separated by commas or a specified separator string. If the array has only one item, then that item will be returned without using the separator.*

~~~ javascript
const months = ['Jan', 'Feb', 'Mar']

elements.join();
> 'Jan,Feb,Mar'

elements.join('');
> 'JanFebMar'

elements.join(' ');
> 'Jan Feb Mar'

elements.join('-');
> 'Jan-Feb-Mar'

const days = ['Sun']
days.join('-')
> 'Sun'
~~~

### `Array.prototype.includes()`

*The `includes()` method determines whether an array includes a certain value among its entries, returning `true` or `false` as appropriate.*

~~~ javascript
const months = ['Jan', 'Feb', 'Mar', ['Jan']]

months.includes('Jan')
> true

months.includes(['Jan'])
> false
~~~

## References
* "Array.prototype.includes() - Web APIs \| MDN." *MDN Web Docs*, Mozilla Web Network, 3 Aug. 2021, <[`developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/includes`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/includes)>.
* "Array.prototype.join() - Web APIs \| MDN." *MDN Web Docs*, Mozilla Web Network, 3 Aug. 2021, <[`developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/join`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/join)>.
* "Array.prototype.push() - Web APIs \| MDN." *MDN Web Docs*, Mozilla Web Network, 13 Dec. 2021, <[`developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/push`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/push)>.
* "Array.prototype.splice() - Web APIs \| MDN." *MDN Web Docs*, Mozilla Web Network, 10 Dec. 2021, <[`developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/splice`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/splice)>.
* Edwards, Benj. "What Is the Konami Code, and How Do You Use It?" *How-To Geek*, 24 Aug. 2021, <[`howtogeek.com/659611/what-is-the-konami-code-and-how-do-you-use-it`](www.howtogeek.com/659611/what-is-the-konami-code-and-how-do-you-use-it)>.
* "Key Sequence Detection (KONAMI CODE)" *wesbos.com*, uploaded by Wes Bos, 8 Dec. 2016, <[`javascript30.com`](https://javascript30.com/)>.