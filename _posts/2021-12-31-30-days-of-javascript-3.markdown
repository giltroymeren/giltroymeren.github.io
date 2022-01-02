---
layout: post
title:  "Notes on Day 3 of 30 Days of JavaScript"
date:   2021-12-31
categories: notes series javascript css javascript30
---

This is my first time to encounter variables in native CSS. The lesson was exciting and I learned a lot!

## Table of Contents
* TOC
{:toc}

## Lessons Learned

### CSS variables

Also know as "Custom Properties", these enables the use of variables to assign CSS styles.
Fortunately it is widely implemented now in [most modern browsers](https://caniuse.com/css-variables).

For organizational purpose, the `:root` pseudo-class can be used to declare common styles.

~~~~ css
:root {
  --boxPadding: 16px;
  --baseColor: #000000;
  --accentColor: #FCFCFC;
  ...
  --myCustomName: ...  
}
~~~~

and these can be used alongside standard CSS inside style blocks with the use of `var()`:

~~~~ css
.box-container {
  display: block;
  padding: var(--boxPadding);
  border-radius: 8px;
  border: 1px solid var(--accentColor);
  ...
}
~~~~

Lastly, since this is CSS, their values can also be overriden if re-declared or re-assigned afterwards inside the file.

### `NodeList.prototype.forEach`

I know `forEach` is available for array objects but I didn't know the `NodeList` object had this in its prototype. 
This is covenient for skipping transforming into an array.

~~~~ javascript
const spanList = document.querySelectorAll('span')
// NodeList(5) [span, span, span.cm-variable, span.cm-property, span.cm-variable]

spanList.forEach(span => console.log(span))
// <span>...</span>
~~~~

### `HTMLElement: change` event

This event is available for common form elements such as `<input>` and `<textarea>` wherein it listens for the changes in the elements' values.

~~~ html
<input type="email" name="email" />
<p id="info">Logged in as <span></span><p>
~~~

~~~ javascript
const email = document.querySelector('input[type="email"]')
const message = document.querySelector(p#info span)

const showMessage = (event) => {
  message.textContent = event.target.value
}

email.addEventListener('change', showMessage)
~~~

### `Element: mouseover` event

This event fires when a cursor is within the scope of an element or its children.

### `HTMLElement.dataset`

We can declare custom data attributes inside DOM elements that are accessible using the `dataset` keyword.
Accessing this would return a `DOMStringMap` object.

~~~ html
<textarea rows="1" data-show="false" data-test="list-page" data-original-title="null"></textarea>
~~~

~~~ javascript
console.log(document.querySelector('textarea'))

{
  "show": "false",
  "test": "list-page",
  "originalTitle": "null"
}
~~~

### `CSSStyleDeclaration.setProperty()`

I didn't know this generalized method existed because all I did before was to set the individual styles as is!

In my defense, I *rarely* set CSS properties using JS directly because if we can do it through setting selector then this would be the last resort of implementation.

~~~ javascript
const story = document.querySelector('p.story')
// These both set the text color to red 
story.style.color = 'red'
story.style.setProperty('color', 'red')
~~~

## References
* "CSSStyleDeclaration.setProperty() - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 23 Sept. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/CSSStyleDeclaration/setProperty`](https://developer.mozilla.org/en-US/docs/Web/API/CSSStyleDeclaration/setProperty)>.
* "HTMLElement: change Event - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 14 Sept. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/HTMLElement/change_event`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/change_event)>.
* "HTMLElement.dataset - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 14 Sept. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/HTMLElement/dataset`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/dataset)>.
* "Element: mouseover Event - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 14 Sept. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/Element/mouseover_event`](https://developer.mozilla.org/en-US/docs/Web/API/Element/mouseover_event)>.
* "NodeList.prototype.ForEach() - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 14 Sept. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/NodeList/forEach`](https://developer.mozilla.org/en-US/docs/Web/API/NodeList/forEach)>.
* "Playing with CSS Variables and JS." *wesbos.com*, uploaded by Wes Bos, 8 Dec. 2016, <[`javascript30.com`](https://javascript30.com/)>.
* "Using CSS Custom Properties (Variables) - CSS: Cascading Style Sheets \| MDN." *MDN Web Docs*, Mozilla Developer Network, 27 Nov. 2021, <[`developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties`](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties)>.