---
layout: post
title:  "Notes on asynchronous JavaScript programming"
date:   2020-07-25
categories: notes series javascript async
---

I recently read up on JavaScript `Promises` and Sasha Vodnik's ["JavaScript: Async"](https://www.linkedin.com/learning/javascript-async/implementing-smart-asynchronous-code?u=67698794) helped me better understand its history and current usage.

## Table of Contents
* TOC
{:toc}

## Synchronous and asynchronous programming

There are two ways code can be executed in JS:

1. Synchronous refers to running each line of code and waiting for them to finish before the next. This idle time is also refered to as "blocking".

    This is common for steps that need to be executed chronologically and depends on its predecessor.

2. Asynchronous on the other hand does not wait for other lines of code to finish execution. They are immediately executed and JS provides ways to catch their respective responses after they're done.

    Async code do not need others to execute independently hence the convenience of no blocking occurrences in its flow.
    <div>
    <?xml version="1.0" encoding="UTF-8"?>
    <svg viewBox="0 0 904 230" version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" aria-labelledby="title">
    <title id="title" lang="en">Diagrams for synchronous and asynchronous processing</title>
    <desc>Two diagrams showing queued execution of synchronous code versus immediate execution of asynchornous lines of code.</desc>
    <g id="Page-1" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
        <g id="Sync">
            <rect id="Rectangle-Copy-4" fill="#D8D8D8" x="23" y="200" width="335" height="30"></rect>
            <rect id="Rectangle-Copy-3" fill="#D8D8D8" x="23" y="150" width="335" height="30"></rect>
            <rect id="Rectangle-Copy-2" fill="#D8D8D8" x="23" y="100" width="335" height="30"></rect>
            <rect id="Rectangle-Copy" fill="#D8D8D8" x="23" y="50" width="335" height="30"></rect>
            <rect id="Rectangle" fill="#D8D8D8" x="23" y="0" width="335" height="30"></rect>
            <g id="Arrows" transform="translate(365.000000, 23.000000)" fill="#979797" fill-rule="nonzero">
                <path id="Line-Copy-3" d="M29,174.613803 C29,183.119531 25.4319636,189.264731 18.4046244,192.828742 L19.889959,201.083479 L-0.492097291,195.098349 L16.5252405,182.383782 L17.8475329,189.725364 C23.3138937,186.640235 26,181.666735 26,174.613803 C26,162.3682 17.7713152,156.382086 0.546529227,156.511688 L0.0218115983,156.517487 L-1.47802981,156.539299 L-1.52165301,153.539616 L-0.0218115983,153.517804 C19.0624271,153.24027 29,160.32166 29,174.613803 Z"></path>
                <path id="Line-Copy-2" d="M29,121.603999 C29,130.109727 25.4319636,136.254927 18.4046244,139.818938 L19.889959,148.073675 L-0.492097291,142.088545 L16.5252405,129.373978 L17.8475329,136.71556 C23.3138937,133.630431 26,128.656931 26,121.603999 C26,109.358396 17.7713152,103.372282 0.546529227,103.501884 L0.0218115983,103.507683 L-1.47802981,103.529495 L-1.52165301,100.529812 L-0.0218115983,100.508 C19.0624271,100.230466 29,107.311856 29,121.603999 Z"></path>
                <path id="Line-Copy" d="M29,68.6039992 C29,77.1097272 25.4319636,83.2549272 18.4046244,86.8189384 L19.889959,95.0736753 L-0.492097291,89.0885452 L16.5252405,76.3739783 L17.8475329,83.7155596 C23.3138937,80.6304308 26,75.6569311 26,68.6039992 C26,56.3583961 17.7713152,50.3722817 0.546529227,50.5018839 L0.0218115983,50.507683 L-1.47802981,50.5294946 L-1.52165301,47.5298118 L-0.0218115983,47.5080002 C19.0624271,47.2304657 29,54.3118564 29,68.6039992 Z"></path>
                <path id="Line" d="M29,19.6039992 C29,28.1097272 25.4319636,34.2549272 18.4046244,37.8189384 L19.889959,46.0736753 L-0.492097291,40.0885452 L16.5252405,27.3739783 L17.8475329,34.7155596 C23.3138937,31.6304308 26,26.6569311 26,19.6039992 C26,7.35839607 17.7713152,1.37228168 0.546529227,1.50188385 L0.0218115983,1.50768301 L-1.47802981,1.52949461 L-1.52165301,-1.47018821 L-0.0218115983,-1.49199981 C19.0624271,-1.76953432 29,5.31185639 29,19.6039992 Z"></path>
            </g>
            <g id="Labels" transform="translate(0.000000, 9.000000)" fill="#999999" font-family="OpenSans-Bold, Open Sans" font-size="16" font-weight="bold">
                <text id="5">
                    <tspan x="0.43359375" y="212">5</tspan>
                </text>
                <text id="4">
                    <tspan x="0.43359375" y="162">4</tspan>
                </text>
                <text id="3">
                    <tspan x="0.43359375" y="112">3</tspan>
                </text>
                <text id="2">
                    <tspan x="0.43359375" y="62">2</tspan>
                </text>
                <text id="1">
                    <tspan x="0.43359375" y="17">1</tspan>
                </text>
            </g>
        </g>
        <g id="Async" transform="translate(464.000000, 0.000000)">
            <rect id="Rectangle-Copy-4" fill="#D8D8D8" x="23" y="200" width="335" height="30"></rect>
            <rect id="Rectangle-Copy-3" fill="#D8D8D8" x="23" y="150" width="335" height="30"></rect>
            <rect id="Rectangle-Copy-2" fill="#D8D8D8" x="23" y="100" width="335" height="30"></rect>
            <rect id="Rectangle-Copy" fill="#D8D8D8" x="23" y="50" width="335" height="30"></rect>
            <rect id="Rectangle" fill="#D8D8D8" x="23" y="0" width="335" height="30"></rect>
            <g id="Arrows" transform="translate(367.000000, 17.000000)" fill="#979797" fill-rule="nonzero">
                <path id="Line-2" d="M22,-9 L41,0.5 L22,10 L22,2 L-1,2 L-1,-1 L22,-1 L22,-9 Z"></path>
                <path id="Line-2-Copy-5" d="M22,191 L41,200.5 L22,210 L22,202 L-1,202 L-1,199 L22,199 L22,191 Z"></path>
                <path id="Line-2-Copy-4" d="M22,141 L41,150.5 L22,160 L22,152 L-1,152 L-1,149 L22,149 L22,141 Z"></path>
                <path id="Line-2-Copy-3" d="M22,91 L41,100.5 L22,110 L22,102 L-1,102 L-1,99 L22,99 L22,91 Z"></path>
                <path id="Line-2-Copy" d="M22,41 L41,50.5 L22,60 L22,52 L-1,52 L-1,49 L22,49 L22,41 Z"></path>
            </g>
            <g id="Ellipses" transform="translate(0.000000, 9.000000)" fill="#999999" font-family="OpenSans-Bold, Open Sans" font-size="16" font-weight="bold">
                <text id="5">
                    <tspan x="0.43359375" y="212">5</tspan>
                </text>
                <text id="4">
                    <tspan x="0.43359375" y="162">4</tspan>
                </text>
                <text id="3">
                    <tspan x="0.43359375" y="112">3</tspan>
                </text>
                <text id="2">
                    <tspan x="0.43359375" y="62">2</tspan>
                </text>
                <text id="1">
                    <tspan x="0.43359375" y="17">1</tspan>
                </text>
            </g>
            <g id="Labels" transform="translate(426.000000, 6.000000)" fill="#999999" font-family="OpenSans-Bold, Open Sans" font-size="16" font-weight="bold">
                <text id="…">
                    <tspan x="0.16015625" y="212">…</tspan>
                </text>
                <text id="…">
                    <tspan x="0.16015625" y="162">…</tspan>
                </text>
                <text id="…">
                    <tspan x="0.16015625" y="112">…</tspan>
                </text>
                <text id="…">
                    <tspan x="0.16015625" y="62">…</tspan>
                </text>
                <text id="…">
                    <tspan x="0.16015625" y="17">…</tspan>
                </text>
            </g>
        </g>
    </g>
    </svg>
    </div>

## Ways to implement asynchronous programming in JS

### `XMLHttpRequest`

This is the earliest way to communicate with server API with support starting from IE7. It is closely used with the AJAX (asynchronouse JavaScript and XML) technology.

~~~ javascript
let httpRequest = new HttpRequest();
httpRequest.open('GET', url);

httpRequest.onload = () => {
    if (httpRequest.status === 200) {
        successHandler(httpRequest.responseText);
    } else {
        failHandler(httpRequest.status)
    }
}

httpRequest.send();
~~~

In here I checked the response status in order to implement handling of successful and failed communication.

### Promises

Starting in ES6, the `Promises` was shipped to browsers that enabled a new way to handle asynchronous acceptance (`resolve`) or rejection (`reject`) of responses.

~~~ javascript
function get(url) {
    return new Promise((resolve, reject) => {
        ...
        resolve(response);
        ...
        reject(response);
    })
}

get(url)
    .then(response => {
        successHandler(response);
    })
    .catch(status => {
        failHandler(status);
    })
    .finally(() => {
        ...
    });
~~~

Before I thought that the `resolve` and `reject` objects were direct equivalents of the `successHandler` and `failHandler` respectively from my examples and that the latter *should* substitute the first in code. Fortunately I now know better.

#### Chainable instance methods

- `then()`
    - handles `resolve` cases and can be chained to other `then()`s to pass response objects.
- `catch()`
    - handles `reject` cases.
- `finally()`
    - executes after the first two regardless of the result.

#### `Promise.all`

In real projects we need to deal with multiple fetches of data in bulk! Good thing `Promise.all` is available for use.

It just takes an array of `Promise` objects and will resolve them in the chains.

~~~ javascript
Promise.all(urlArray.map(url => get(url)))
    .then(responses => {
        return responses.map(response =>
            successHandler(response);
        );
    })
    .then(data => {
        ...
    })
    .catch(status => {
        failHandler(status);
    })
    .finally(() => {
        ...
    });
~~~

- Expect that it will return an array of responses and will need specific case handling based on your logic.
- In my example there is a second `then()` with the `data` parameter. This represents an array of the individually-processed response objects in the initial `then()`.

### `async`/`await`

These two keywords are applied to functions that deal with asynchronous code or basically has `Promise`s inside.

- `async` is attached before the `function` declaration.
- `await` is attached before methods that return a `Promise` to be resolved, usually during assignment. This keyword is only permitted in a method that is declared `async` and can be used multiple times.

> In order to not mix `Promise.all` here I declared the individual `get()` calls to show only the two keywords in use.

~~~ javascript
(async function() {
    try {
        let responses = [];
        responses.push(await get(urlArray[0]));
        ...
        responses.push(await get(urlArray[n]));

        let data = response.map(response =>
            successHandler(response);
        );

        ...
    } catch (status) {
        failHandler(status);
    } finally {
        ...
    }
})();
~~~

Instead of the chained methods in `Promise`, the familiar `try`, `catch` and `finally` pattern is used here.

#### Dealing with array comprehension

We can mix these keywords with `Promise.all()` to take advantage of the new array methods! The code before can be simplified to:

~~~ javascript
(async function() {
    try {
        let responses = await Promise.all(urlArray.map(url => get(url)));

        let literals = responses.map(response => {
            return successHandler(response);
        });
        ...
~~~

I initially tried to do the following code but the `push` method is synchronous so it will fail to wait for the `get()` process.

~~~ javascript
urlArray.map(url => {
    responses.push(await get(url));
});
~~~



## References
* Ajedi32. “Use Async Await with Array.map.” Stack Overflow, Stack Overflow, 19 Oct. 2016, <[`https://stackoverflow.com/a/40140562/3314071`](https://stackoverflow.com/a/40140562/3314071)>.
* Elliott, Eric. “Master the JavaScript Interview: What Is a Promise?” Medium, JavaScript Scene, 2 July 2019, <[`https://medium.com/javascript-scene/master-the-javascript-interview-what-is-a-promise-27fc71e77261`](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-a-promise-27fc71e77261)>.
* Kantor, Ilya. “Async/Await.” The Modern JavaScript Tutorial, The Modern JavaScript Tutorial, 3 Feb. 2020, <[`https://javascript.info/async-await`](https://javascript.info/async-await)>.
* “Promise.prototype.catch().” MDN Web Docs, Mozilla Development Network, 2020, <[`https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/catch`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/catch)>.
* Vodnik, Sasha. “JavaScript: Async.” LinkedIn Learning, LinkedIn Learning, 14 Oct. 2019, <[`https://linkedin.com/learning/javascript-async/`](https://linkedin.com/learning/javascript-async/)>.
* “XMLHttpRequest.” MDN Web Docs, Mozilla Development Network, 2020, <[`https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest`](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest)>.
