---
layout: post
title:  "Notes on Day 16 of 30 Days of JavaScript"
date:   2022-01-13
categories: notes series javascript30
---

## Table of Contents
* TOC
{:toc}

## Lessons Learned

### `HTMLElement.contenteditable`

The `contentEditable` attribute specifies if the element is editable or not. Its default value is `'true'` and can also be `'false'` and `'inherit'`.

~~~ html
<h1 contenteditable>This is the title</h1>
~~~

This does not affect `input` and `textarea` elements regardless if set to false. You can still enter text to them. Only the `disabled` attribute would prevent entering data.

In terms of inheritance, it will only take effect if the `'inherit'` value is explicitly set to a children.

~~~ html
<div contenteditable="false">
  <p>This paragraph is not editable</p>
  <p contenteditable="inherit">This paragraph is not editable</p>
  <p contenteditable="true">This paragraph is editable</p>
</div>
~~~

## References
* "CSS Text Shadow Mouse Move Effect" *wesbos.com*, uploaded by Wes Bos, 8 Dec. 2016, <[`javascript30.com`](https://javascript30.com/)>.
* "HTMLElement.contentEditable - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 15 Sep. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/HTMLElement/contentEditable`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/contentEditable)>.