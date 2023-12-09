---
layout: post
title: "Notes on Day 1 of 2023 AdventJS Challenge"
date: 2021-12-29
categories: notes series javascript adventjs2023
---

I recently attended [React Day Berlin](https://reactday.berlin/) and [Miguel Ángel Durán](https://github.com/midudev) gave a shoutout to the [AdventJS challenge](https://adventjs.dev/) which interested me to try and as an excuse to add more posts in this blog of mine.

![Miguel Ángel Durán](/images/posts/2023-12-09-notes-on-day-1-of-2023-adventjs-challenge/react-day-berlin-miguel-angel-duran.jpeg){:class="img-responsive"}

## Table of Contents

- TOC
  {:toc}

## Challenge

[Day 1](https://adventjs.dev/challenges/2023/1) starts with a bang with a duplicate search problem:

> Find the first identification number that has been repeated, **where the second occurrence has the smallest index**!
> In other words, if there is more than one repeated number, you must return the number whose second occurrence appears first in the list. If there are no repeated numbers, return -1.

I have encountered a lot of these integrated in my JS bootcamps and several frontend developer application exams and I think this is a classic staple for any developer to solve.

## Lessons Learned

### `Array.prototype.filter()`

I decided to use good old `filter` to extract all elements that are not unique in the array.

Normally in order to get all unique elements in an array, we use the criteria to just get all items with the same index as the current one.

~~~ javascript
myArray.filter((item, index) => myArray.indexOf(item) === index)
~~~

The way `indexOf` works is helpful here because it only returns the index of the first existence of the given item if available, otherwise `-1`.

So in reverse we get the duplicated items instead:

~~~ javascript
myArray.filter((item, index) => myArray.indexOf(item) !== index)
~~~

### `Array.prototype.indexOf()`

It is an important tool here as mentioned because it always returns the first instance of an item in an array.

## References

- "Array.protoype.filter() - JavaScript \| MDN." _MDN Web Docs_, Mozilla Developer Network, 9 Dec. 2023, <[`developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter)>.
- "Array.protoype.indexOf() - JavaScript \| MDN." _MDN Web Docs_, Mozilla Developer Network, 9 Dec. 2023, <[`developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/indexOf`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/indexOf)>.