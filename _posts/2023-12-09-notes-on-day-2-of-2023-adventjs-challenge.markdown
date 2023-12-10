---
layout: post
title: "Notes on Day 2 of 2023 AdventJS Challenge"
date: 2023-12-09
categories: notes series javascript adventjs2023 typescript
---

[Day 2](https://adventjs.dev/en/challenges/2023/2) surprised me because it continued the Christmas theme and the gift-making premise of the first one and I wouldn't have it any other way. I am now expecting the rest of the challenges to build on top of this.

## Table of Contents

- [Table of Contents](#table-of-contents)
- [Challenge](#challenge)
  - [Solution](#solution)
- [Lessons Learned](#lessons-learned)
  - [`Array.prototype.concat()`](#arrayprototypeconcat)
  - [`Array.prototype.every()`](#arrayprototypeevery)
  - [`Array.prototype.split()`](#arrayprototypesplit)
  - [Declaring `export {}`](#declaring-export-)
  - [Using `interface` and `type` in TypeScript](#using-interface-and-type-in-typescript)
- [References](#references)

## Challenge

Today's topic is related to the previous day's string manipulation problem wherein given a random string of alphabet characters or "materials", we need to determine if a "gift" or an actual word (which are Spanish *en esto caso hoy*) can be spelled using them.

### Solution

My solution was simple: split each *gift*, sort alphabetically and compare with the characters of *materials* that is also split alphabetically. If all characters of said *gift* are found, then it is doable in Santa's workshop.

~~~ javascript
function manufacture(gifts: Array<string>, materials: string) {
  const materialsArr = materials.split("").sort();

  const results: Array<string> = [];
  for (let i = 0; i < gifts.length; i++) {
    const gift = gifts[i].split("").sort();
    if (gift.every((letter) => materialsArr.includes(letter))) {
      results.push(gifts[i]);
    }
  }

  return results;
}
~~~

## Lessons Learned

I had to revisit some array prototypes today which I think are good presents for a developer!

### `Array.prototype.concat()`

I am always using `push` to add elements into the end arrays and looked up other ways to do it out of curiosity. `concat` does the same, essentially a typical stack push, but it returns a new array without changing the original array.

### `Array.prototype.every()`

The comparison array protoypes `every` and `some` are really helpful in some cases in daily work and in today's problem, I used the first to keep track if all characters of a *gift* exist in the *materials*.

### `Array.prototype.split()`

We all use `split` in our coding challenges' early array problems and today is not an exception, at least for my solution to the problem.

### Declaring `export {}`

I have been using TypeScript a lot at work and I like the level of "strictness" it gives the developer and so I decided to implement my solutions in TS.

I encountered the following error in my Day 2 TS file when declaring the `tests` variable:

> Cannot redeclare block-scoped variable 'tests'.ts(2451)

This is because I also have the same variable declared in the Day 1 file and what solved it is adding an empty `export` object at the end of each TS file.

I was only running each TS file in the terminal so I believe that's simple and perfect enough for my case aside from the other compiler instructions available.

### Using `interface` and `type` in TypeScript

In addition to my TS adventures, I wanted to declare a type (or an interface?) for today's inputs and remembered which between `type` and `interface` is better at least for my simple file?

[Hochel's extensive explanation](https://medium.com/@martin_hotell/interface-vs-type-alias-in-typescript-2-7-2a8f1777af4c) stated what I needed: type is a unique entity while interface can be overriden with the same name.

I know in the end its based on preferences and consistency in the codebase but it's nice to read back on the idiosyncrasies of reserved words we use when programming.

## References

- aphilas. "Cannot Redeclare Block Scoped Variable." *Stackoverflow*, 18 Jun. 2018, <[`stackoverflow.com/a/50913569`](https://stackoverflow.com/a/50913569)>. Accessed 9 Dec. 2023.
- "Array.protoype.concat() - JavaScript \| MDN." _MDN Web Docs_, Mozilla Developer Network, 9 Dec. 2023, <[`developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/concat`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/concat)>.
- "Array.protoype.every() - JavaScript \| MDN." _MDN Web Docs_, Mozilla Developer Network, 9 Dec. 2023, <[`developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/every`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/every)>.
- "Array.protoype.split() - JavaScript \| MDN." _MDN Web Docs_, Mozilla Developer Network, 9 Dec. 2023, <[`developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/split`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/split)>.
- Hochel, Martin. "Interface Vs Type Alias in TypeScript 2.7." *Medium*, 12 Mar. 2018, <[`medium.com/@martin_hotell/interface-vs-type-alias-in-typescript-2-7-2a8f1777af4c`](https://medium.com/@martin_hotell/interface-vs-type-alias-in-typescript-2-7-2a8f1777af4c)>. Accessed 9 Dec. 2023.
- Olawanle, Joel. "Push into an Array in JavaScript – How to Insert an Element into an Array in JS." *FreeCodeCamp*, 18 Jul. 2022, <[`freecodecamp.org/news/how-to-insert-an-element-into-an-array-in-javascript`](https://www.freecodecamp.org/news/how-to-insert-an-element-into-an-array-in-javascript/)>. Accessed 9 Dec. 2023.
- Piros, Tamas. "Cannot Redeclare Block-scoped Variable 'name'." *Fullstack Developer Academy*, 9 Jun. 2018, <[`web.archive.org/web/20180609155906/http://fullstack-developer.academy/cannot-redeclare-block-scoped-variable-name/`](https://web.archive.org/web/20180609155906/http://fullstack-developer.academy/cannot-redeclare-block-scoped-variable-name/)>. Accessed 9 Dec. 2023.