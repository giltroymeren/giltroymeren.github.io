---
layout: post
title:  "Notes on Day 4 of 30 Days of JavaScript"
date:   2022-01-01
categories: notes series javascript 30_days_of_js
---

I believe I am well-versed with the latest JS array methods but I managed to learn new techniques today!

## Table of Contents
* TOC
{:toc}

## Lessons Learned

~~~ javascript
const people = [
  'Bernhard, Sandra', 'Bethea, Erin', 'Becker, Carl', 'Bentsen, Lloyd',
  ...
  'Birrell, Augustine', 'Blair, Tony', 'Beecher, Henry', 'Biondo, Frank'
];
~~~

### `String.prototype.localeCompare()`

The 7th challenge in the activity was to sort the items in the `people` array by last name:

My first thought was to use a custom function for `Array.prototype.sort()` to compare the last names similar to integers and I found the `localeCompare` method.

I kept getting the following error message that frustrated me:

~~~
"TypeError: a.localCompare is not a function
~~~

and I even tried to check the type of the names and they are all `String`s! I realized it was "locale" with an *e* and not "local".

Fortunately it worked as

~~~ javascript
peopple.sort((a, b) => a.localeCompare(b.toString()))
// 
~~~

Lastly I tried using `sort()` as it is without any callback and it worked for some reason.
But I do not recommend this because it might just be my browser. I will just stick to the callback way at least we are sure of the operations applied.

### `Array.prototype.reduce()` accumulator for unique items

The last challenge was to return the occurrences of elements in an array (the result's data structure wasn't stated).

~~~ javascript
const data = ['car', 'car', 'truck', 'truck', 'bike', 'walk', 'car', 
  'van', 'bike', 'walk', 'car', 'van', 'car', 'truck' ];
~~~

My initial plan was to `map` over the items and create a `Set` for uniqueness of elements then save the number occurrences based on it. The solution was far more elegant than my approach because it just used the `reduce` method:

~~~ javascript
data.reduce((obj, item) => {
  if(!obj[item]) obj[item] = 0
  obj[item]++
  return obj
}, {})
~~~

I never thought that the second parameter (the initial value) would be useful in this approach.
If we try to run the same code without the empty object, it will just return `"car"` ad the first element.

## References
* "Array Cardio Day 1." *wesbos.com*, uploaded by Wes Bos, 8 Dec. 2016, <[`javascript30.com`](https://javascript30.com/)>.
* "Sort array by firstname (alphabetically) in Javascript." *Stack Overflow*, stackoverflow.com, 15 July 2011, <[`stackoverflow.com/questions/6712034/sort-array-by-firstname-alphabetically-in-javascript/45544166#45544166`](https://stackoverflow.com/a/45544166).
* "String.prototype.localeCompare() - JavaScript \| MDN.” *MDN Web Docs*, Mozilla Developer Network, 20 July 2021, <[`developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/localeCompare`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/localeCompare)>.