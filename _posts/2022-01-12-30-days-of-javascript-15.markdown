---
layout: post
title:  "Notes on Day 15 of 30 Days of JavaScript"
date:   2022-01-12
categories: notes series javascript javascript30
---

## Table of Contents
* TOC
{:toc}

## Lessons Learned

### Event delegation
Wes used a nice analogy for this concept: "responsible parents and negligent children". Basically we delegate a *single* event listener for a group of items to their common direct parent.

#### Problem

In the example, checkboxes are added to the DOM as the user submits an entry. However we cannot attach an event listener to them as they are created because there are better ways to handle this problem. 

~~~ html
<input type="checkbox" name="task" data-id="task-1" />
<input type="checkbox" name="task" data-id="task-2" />
...
<input type="checkbox" name="task" data-id="task-100" />
~~~

#### Solution

We can group the checkboxes in a container an set the event listener to this element, or their parent. Usually input elements are enclosed in a `form` so we have:

~~~ html
<form id="task-form">
  <input type="checkbox" name="task" data-id="task-1" />
  <input type="checkbox" name="task" data-id="task-2" />
  ...
  <input type="checkbox" name="task" data-id="task-100" />
</form>
~~~

and we can attach a single listener to the parent and just set a general filter to select checkboxes via the `event.target` attribute:

~~~ javascript
function toggleCheckbox(event) {
  if(event.target.matches('input[type=checkbox']))
    // do something
}

document.getElementById('task-form').addEventListener('click', toggleCheckbox)
~~~

This pattern is ideal for groups of DOM elements that are created dynamically and/or need time before being rendered to the screen.

### `Element.matches()` & `event.target.is()`

*The `matches()` method checks to see if the `Element` would be selected by the provided `selectorString` -- in other words -- checks if the element "is" the selector*.

### `this` binding inside methods

While doing the task I mindlessly chose to declare the method as arrow function because it's what I'm used to. 

#### Problem

I was following the task when the following code appeared inside the method:

~~~ javascript
const item = this.querySelector('input[name="item"]')
~~~

After execution, it didn't work and an error occurred:

~~~ 
Uncaught TypeError: this.querySelector is not a function
~~~

#### Solution

I forgot that arrow functions have *limitations* and it includes <u>no binding to the `this` keyword</u>. The traditional `function`-declared methods have. 

~~~ javascript
function addItem(e) {
  e.preventDefault()
  const text = (this.querySelector('input[name=task]')).value
  ...
}
~~~

Its value is represented by a *`DOMString` containing the HTML serialization of the element's descendants*.

### `Element.innerHTML`

*The `Element` property `innerHTML` gets or sets the HTML or XML markup contained within the element.*

~~~ javascript
const container = document.querySelector('container')

const content = container.innerHTML

const.innerHTML = `<p>Hello World</p>`
~~~

### `Storage.getItem()` & `Storage.setItem()`

`getItem()` and `setItem()` are used to access the `Storage` interface of modern browsers to set and retrieve data.

- `setItem(key, value)` takes both parameters as `String` type and adds to the `Storage`
- `getItem(key)` will return the value if the key exists otherwise `null`

~~~ javascript
localStorage.setItem('message', 'Hello World!')
localStorage.getItem('message')
> 'Hello World!'
~~~

- `setItem()` overwrites its value if called to the same key

~~~ javascript
localStorage.setItem('message', 'Hello World!')
localStorage.setItem('message', 'Good day!')
localStorage.getItem('message')
> 'Good day!'
~~~

#### Behavior with arrays

Arrays with primitive types (string, number) as content will be converted to strings and stored. Upon access, they will all be concatenated with commas.

~~~ javascript
// string
localStorage.setItem('arr', ['a', 'b', 'c'])
localStorage.getItem('arr')
> 'a,b,c'

localStorage.setItem('arr', ['Apple',' Banana', 'Carrot'])
localStorage.getItem('arr')
> 'Apple,Banana,Carrot'

// number
localStorage.setItem('arr', [1, 2, 3, 4, 5])
localStorage.getItem('arr')
> '1,2,3,4,5'

// boolean
localStorage.setItem('arr', [true, false, true])
localStorage.getItem('arr')
> 'true,false,true'

// null and undefined
localStorage.setItem('arr', [null, undefined, null, undefined])
localStorage.getItem('arr')
> ',,,'
~~~

#### Behavior with objects

Regardless if the value is an object or an array of objects, each element will be stored as `"[object Object]"`.

~~~ javascript
// object
localStorage.setItem('obj', { name: 'Abc', id: 1 })
localStorage.getItem('obj')
> '[object Object]'

// array of objects
localStorage.setItem('obj', [{ name: 'Abc', id: 1 }, { name: 'Xyz', id: 2 }])
localStorage.getItem('obj')
> '[object Object],[object Object]'
~~~

### `JSON.stringify()` & `JSON.parse()`

In relation to accessing the contents of the `Storage` object, we can use these methods to properly store and retrieve object types.

Actually we can store objects as JSON strings and parse them after retrieval.

~~~ javascript
// array of objects
const data = JSON.stringify([{ name: 'Abc', id: 1 }, { name: 'Xyz', id: 2 }])
localStorage.setItem('data', data)

JSON.parse(localStorage.getItem('data'))
> '[{ name: "Abc", id: 1 }, { name: "Xyz", id: 2 }]'
~~~

## References
* "Arrow function expressions - JavaScript \| MDN." *MDN Web Docs*, Mozilla Developer Network, 1 Dec. 2021, <[`developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)>.
* "Element.innerHTML - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 8 Dec. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/Element/innerHTML`](https://developer.mozilla.org/en-US/docs/Web/API/Element/innerHTML)>.
* "Element.matches - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 19 Sep. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/Element/matches`](https://developer.mozilla.org/en-US/docs/Web/API/Element/matches)>.
* Kantor, Ilya. "Event Delegation." *javascript.info*, 12 Dec. 2021, <[`javascript.info/event-delegation`](https://javascript.info/event-delegation)>.
* "LocalStorage and Event Delegation" *wesbos.com*, uploaded by Wes Bos, 8 Dec. 2016, <[`javascript30.com`](https://javascript30.com/)>.
