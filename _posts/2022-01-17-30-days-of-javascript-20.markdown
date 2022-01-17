---
layout: post
title:  "Notes on Day 20 of 30 Days of JavaScript"
date:   2022-01-17
categories: notes series javascript javascript30 speech-recognition
---

## Table of Contents
* TOC
{:toc}

## Lessons Learned

### `SpeechRecognition`

This interface, a part of the Web Speech API, is *the controller interface for the recognition service*.

Some of its properties are the following:

- `SpeechRecognition.lang` - sets the language of the current `SpeechRecognition` instance.
- `SpeechRecognition.interimResults` - accepts boolean values and *controls whether interim results should be returned* based on the setting.
- `SpeechRecognition.continuous` - *controls whether continuous results are returned for each recognition, or only a single result*.

~~~ javascript
const speechRecognition = new SpeechRecognition();
speechRecognition.lang = 'en-US';
speechRecognition.interimResults = true;
speechRecognition.continuous = false;
~~~

### `SpeechRecognitionEvent.results`

This interface *returns a `SpeechRecognitionResultList` object representing all the speech recognition results for the current session*.

~~~ javascript
speechRecognition.addEventListener('results', event => {
  const transcript = Array.from(event.results)
    .map(result => result[0])
    .map(result => result.transcript)
    .join('');

  document.getElementById('transcript').textContent(transcript)
});
~~~

### `Node.textContent`

*The `textContent` property of the `Node` interface represents the text content of the node and its descendants*.

~~~ html
<div>
  <h3>Section title</h3>
  <p>Section <strong>description</strong></p>
</div>
~~~

Accessing its content returns a string or otherwise `null`.

~~~ javascript
console.log(document.querySelector('div').textContent)
> " 
    Section title
   Section description
  "
~~~

The content can also be set through assignment:

~~~ javascript
document.querySelector('div').textContent = "Hello World";
~~~

~~~ html
<div class"container">Hello World</div>
~~~

> `textContent` is different from `innerHTML`. Please refer to [this page](https://developer.mozilla.org/en-US/docs/Web/API/Node/textContent#differences_from_innertext) for some important differences.

## References
* "Node.textContent - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 8 Nov. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/Node/textContent`](https://developer.mozilla.org/en-US/docs/Web/API/Node/textContent)>.
* "SpeechRecognition - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 17 Dec. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/SpeechRecognition`](https://developer.mozilla.org/en-US/docs/Web/API/SpeechRecognition)>.
* "SpeechRecognitionEvent.results - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 15 Sep. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/SpeechRecognitionEvent/results`](https://developer.mozilla.org/en-US/docs/Web/API/SpeechRecognitionEvent/results)>.
* "Native Speech Recognition" *wesbos.com*, uploaded by Wes Bos, 8 Dec. 2016, <[`javascript30.com`](https://javascript30.com/)>.
