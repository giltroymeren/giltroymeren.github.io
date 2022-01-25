---
layout: post
title:  "Notes on Day 29 of 30 Days of JavaScript"
date:   2022-01-26
categories: notes series javascript javascript30 
---

## Table of Contents
* TOC
{:toc}

## Lessons Learned

### `Document.forms`

A property of the `Document` interface that returns all `<form>` elements in the form of an `HTMLCollection`.

~~~ html
<div>
  <form><input /><button>Send 1</button></form>
  <form><input /><button>Send 2</button></form>
  <form><input /><button>Send 3</button></form>
  <form><input /><button>Send 4</button></form>
  <form><input /><button>Send 5</button></form>
</div>
~~~

~~~ javascript
const forms = document.forms
Array.from(forms).map(form => console.log(form.textContent))

> "Send 1"
  "Send 2"
  "Send 3"
  "Send 4"
  "Send 5"
~~~

### `submit`

This event listener correspons to the `submit` event of `<form>` elements.

~~~ html
<form><input name="name" /><button>Submit</button></form>
~~~

~~~ javascript
const form = document.querySelector("form")
form.addEventListener("submit", (event) => {
  event.preventDefault()
  console.log("Form submitted!")
})
~~~

### `clearInterval`

This method clears and stops a named interval created from a `setInterval()`.

~~~ javascript
const timer = setInterval(() => console.log("Hello"), 1000)
> "Hello"
  "Hello"
  "Hello"
  "Hello" 
  ...

clearInterval(timer)
~~~

## References
* "Countdown Clock" *wesbos.com*, uploaded by Wes Bos, 8 Dec. 2016, <[`javascript30.com`](https://javascript30.com/)>.
* "Document.forms - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 15 Sep. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/Document/forms`](https://developer.mozilla.org/en-US/docs/Web/API/Document/forms)>.
