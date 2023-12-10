---
layout: post
title: "Notes on Day 3 of 2023 AdventJS Challenge"
date: 2023-12-10
categories: notes series javascript adventjs2023 typescript
---

[Day 3](https://adventjs.dev/en/challenges/2023/3) continues the test of string and array manipulation problem-solving skills.

## Table of Contents

- [Table of Contents](#table-of-contents)
- [Challenge](#challenge)
  - [Solution](#solution)
- [Lessons Learned](#lessons-learned)

## Challenge

The third day involves finding discrepancies between two strings and in this case this first occurrence:

> You have the original sequence of original manufacturing steps and the modified modified sequence that may include an extra step or be missing a step.
>
> Your task is to write a function that identifies and returns the first extra step that was added or removed in the manufacturing chain. If there is no difference between the sequences, return an empty string.

### Solution

My initial pseudocode basically simply returns the first character from the base string that does not match the other:
1. Split both strings into characters
     - use longer array as base; otherwise keep original
2. Iterate over the base array's characters
3. Take note of characters that do not match criteria
4. Return first character matched; otherwise empty string

Here's my implementation of said pseudocode with `filter`:

~~~ javascript
function findNaughtyStep(original, modified) {
  const originalArr = original.split("");
  const modifiedArr = modified.split("");

  return originalArr.length > modifiedArr.length
    ? originalArr.filter((char, index) => modifiedArr[index] !== char)[0]
    : modifiedArr.filter((char, index) => originalArr[index] !== char)[0] || "";
}
~~~

## Lessons Learned

I had several versions of the implementation because some of the secret tests failed:
- the function keeps returning all characters that did not match
- the function just returns the difference from the *modified* string (hence my decided in # 1 to check which one's longer)

I know there's more efficient ways and better array prototypes to use and I will keep updating this page after my experiments from this first try to look back at what I have accomplished.
