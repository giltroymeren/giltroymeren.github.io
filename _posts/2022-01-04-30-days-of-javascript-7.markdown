---
layout: post
title:  "Notes on Day 7 of 30 Days of JavaScript"
date:   2022-01-04
categories: notes series javascript javascript30
---

I rarely use the following array methods in my work so it is great to refresh my memory on their capabilities.

## Table of Contents
* TOC
{:toc}

## Lessons Learned

~~~ javascript
const people = [
  { name: 'Wes', year: 1988 },
  { name: 'Kait', year: 1986 },
  { name: 'Irv', year: 1970 },
  { name: 'Lux', year: 2015 }
];
~~~

### `Array.prototype.some()`

Given a condition, it returns `true` if *at least one element* satisfies the condition; otherwise `false`.

~~~ javascript
// Is at least one person 19 or older?
const CURRENT_YEAR = new Date().getFullYear()
const AGE_LIMIT = 19
people.some(a => (CURRENT_YEAR - a.year) >= AGE_LIMIT)
> true
~~~

### `Array.prototype.every()`

Given a condition, it returns `true` if *all elements* satisfy the condition; otherwise `false`.

~~~ javascript
// is everyone 19 or older?
people.every(a => (CURRENT_YEAR - a.year) >= AGE_LIMIT)
> true
~~~

### `Array.prototype.find()`

Given a condition, it returns the *first* element that satisfies the condition; otherwise `undefined`.

~~~ javascript
const comments = [
  { text: 'Love this!', id: 523423, name: 'Abby Z.' },
  { text: 'Super good', id: 823423, name: 'Bart Y.' },
  { text: 'You are the best', id: 2039842, name: 'Chan X.'},
  { text: 'Ramen is my fav food ever', id: 123523, name: 'Day W.' },
  { text: 'Nice Nice Nice!', id: 542328, name: 'Chan X.' },
];

// Find the comment by the name of 'Chan X.'
comments.find(a => a.name === 'Chan X.')

> [object Object] {
  id: 2039842,
  name: "Chan X.",
  text: "You are the best"
}
~~~

The `comments` array has two entries by "Chan X." but `find()` will only return the comment with ID of `2039842` - the first one it encounters. The rest of the elements are ignored.

### `Array.prototype.findIndex()`

Works similar to `find()` but returns the zeroth array index of the element that satisfies the condition. If no element was found then `-1` is returned.

~~~ javascript
// Find the comment by the name of 'Chan X.'
comments.findIndex(a => a.name === 'Chan X.')
> 2

// Find the comment by the name of 'Emma V.'
comments.findIndex(a => a.name === 'Emma V.')
> -1
~~~

## References
* "Array Cardio Day 2." *wesbos.com*, uploaded by Wes Bos, 8 Dec. 2016, <[`javascript30.com`](https://javascript30.com/)>.