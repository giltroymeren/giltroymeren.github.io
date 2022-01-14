---
layout: post
title:  "Notes on Day 17 of 30 Days of JavaScript"
date:   2022-01-14
categories: notes series javascript javascript30
---

## Table of Contents
* TOC
{:toc}

## Lessons Learned

### Regex for multiple text

~~~
replace(/\b(?:this|that|there|those)\b/gi, '')
~~~

- `/.../`: standard Regex syntax
- `\b`: assert position at a word boundary; prevents partial match; exact match
- `(text1|text2)`: match with parameter `'this'`, `'that'` and so on
- `|`: or conditional; note that the text should be near this
- `g` modifier: global match
- `i` modifier: case-insensitive match

~~~ javascript
const text = 'This soup from that store was made over there with those vegetables'
(text.replace(/\b(?:this|that|there|those)\b/gi, '').trim()
> 'soup from  store was made over  with  vegetables'
~~~

## References
* Dib, Firas. "regex101: build, test, and debug regex." *regex101*, >[`regex101.com`](https://regex101.com/)>. Accessed 8 Jan. 2022.
* "Sorting Band Names without articles" *wesbos.com*, uploaded by Wes Bos, 8 Dec. 2016, <[`javascript30.com`](https://javascript30.com/)>.
* The fourth bird. “Replace multiple strings with multiple other strings.” *Stack Overflow*, 3 May 2021, <[`stackoverflow.com/questions/15604140/replace-multiple-strings-with-multiple-other-strings/67374697#67374697`](https://stackoverflow.com/questions/15604140/replace-multiple-strings-with-multiple-other-strings/67374697#67374697)>.
