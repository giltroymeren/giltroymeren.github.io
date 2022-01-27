---
layout: post
title:  "Notes on Day 30 of 30 Days of JavaScript"
date:   2022-01-27
categories: notes series javascript javascript30 
---

I have reached the end of this challenge! Thanks Wes for this wonderful initiative :-)

## Table of Contents
* TOC
{:toc}

## Lessons Learned

### `Node.parentNode`

It returns the immediate parent element `Node` based from the DOM tree structure.

*It also returns `null` if the node has just been created and is not yet attached to the tree*.

~~~ html
<style type="text/css">
  div {
  border: 1px solid #CCC;
  display: block;
  padding: 2rem;
  cursor: pointer;
}

.one { background: pink; }
.two { background: yellow; }
.three { background: orange; }
</style>

<div class="one">
  <div class="two">
    <div class="three">
    </div>
  </div>
</div>
~~~

~~~ javascript
document.querySelector('.three').parentNode
> <div class="two">...</div>

document.querySelector('.two').parentNode
> <div class="one">...</div>

document.querySelector('.one').parentNode
> <body>...</body>
~~~

Getting the parent node of the `<html>` element returns the `document` interface.

~~~ javascript
document.querySelector('html').parentNode
> #document
~~~

## References
* "Node.parentNode - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 8 Nov. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/Node/parentNode`](https://developer.mozilla.org/en-US/docs/Web/API/Node/parentNode)>.
* "Whack A Mole Game" *wesbos.com*, uploaded by Wes Bos, 8 Dec. 2016, <[`javascript30.com`](https://javascript30.com/)>.