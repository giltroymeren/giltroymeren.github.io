---
layout: post
title:  "Real-time Stocks App in React: Part 4"
date:   2022-01-14
categories: notes series javascript typescript react react-hooks websocket lodash
---

The application now works with the minimal requirements to display real-time stocks data. In this post I will discuss how I added a limiter feature for incoming asynchronous data!

For reference, the repository is available at [`github.com/giltroymeren/stocks-app-ts`](https://github.com/giltroymeren/stocks-app-ts).

## Table of Contents
* TOC
{:toc}

### What is "rate limiting"?

Dai perfectly describes this process, in the context of software development, as:

> Rate limiting protects your APIs from inadvertent or malicious overuse by limiting how often each user can call the API. Without rate limiting, each user may make a request as often as they like, leading to “spikes” of requests that starve other consumers. Once enabled, rate limiting can only perform a fixed number of requests per second. A rate limiting algorithm helps automate the process.

In our case, the API, where the stock data comes from, gives fast data albeit asynchronous every second to our `WebSocket` listener. The app needs a way to control these incoming messages in terms of scalability when the number of ISINs being tracked increase further.

### Add a rate limiter to the updates

I decided to implement the rate limiter to the action that updates the stock data in the UI - `updateStock`.

Currently, when the websocket receives responses, it immediately calls `updateStock` and the UI re-rendered:

~~~ javascript
webSocket.current.onmessage = function (event: any) {
  console.info('Receiving message')

  updateStockList(JSON.parse(event.data))
~~~

The theory is to wrap `updateStock` in a rate limiter and execute it given a parameter of let's say one second. Based from my research, the lodash package's `throttle` method fits the bill.

<br />

![Stocks App - Rate limited updateStock method diagram](/images/posts/2022-01-real-time-stocks-app-in-react/rate-limited-update-diagram.svg){:class="img-responsive"}

<br />

Install and add lodash to the dependencies:

~~~ node
npm install --save lodash @types/lodash
~~~

Create a new method to wrap `updateStock` in the `StockProvider` class:

~~~ javascript
const throttledUpdateStockList = useRef(
  throttle((data: IStockItem) => updateStockList(data), 1000)
).current
~~~

`throttle` takes in a function and a time period in milliseconds as parameters. This entire declaration is also wrapped in a `useRef` hook as Rippon suggested to prevent multiple declarations between component renderings.

We can now replace the unwrapped update method in the websocket:

~~~ javascript
webSocket.current.onmessage = function (event: any) {
  console.info('Receiving message')

  throttledUpdateStockList(JSON.parse(event.data))
~~~

### Boostrap for design

This is just a minor detail but I added the [Twitter Bootstrap](https://getbootstrap.com/) design system through CDN.

<br />

![Stocks App - Home Page](/images/posts/2022-01-real-time-stocks-app-in-react/home-page.svg){:class="img-responsive"}

<br />

## Improvements

This is a small app with a ton of limitations (for now) and possibilities. Here are some of my future tasks to add when I have more time:

- add unit tests for the components and state management
- add validation for the `SearchBar` in case of empty or duplicate ISIN
- disable the subscribe/unsubscribe buttons of `StockItems` depending on the process currently running
- further polish the rate limiter to scale with more stocks
- connect to another API to retrieve the entity name of the ISINs
- improve the stock table updates by showing if the amounts were higher, lower or had no changes based on the previous value (could be change in color or addition of "+"/"-" symbols to indicate)
- separate the entire `WebSocket` entity from the provider and put them in their own file
- replace the context with Redux for state management and set the `WebSocket` as a formal middleware

## References
* Dai, Guanlan. "How to Design a Scalable Rate Limiting Algorithm with Kong API." *KongHQ*, 4 Jan. 2022, <[`konghq.com/blog/how-to-design-a-scalable-rate-limiting-algorithm`](https://konghq.com/blog/how-to-design-a-scalable-rate-limiting-algorithm/)>.
* Drori, Ori. "Save data from WebSocket messages only once per 5 seconds." *Stack Overflow*, 29 Jan. 2021, <[`stackoverflow.com/questions/65960931/save-data-from-websocket-messages-only-once-per-5-seconds/65960985#65960985`](https://stackoverflow.com/a/65960985)>.
* Finesse. "How to workaround too often Redux updates in a React application?" *Stack Overflow*, 22 Apr. 2019, <[`stackoverflow.com/questions/55788967/how-to-workaround-too-often-redux-updates-in-a-react-application`](https://stackoverflow.com/q/55788967)>.
* Gigablah. "Uncaught InvalidStateError: Failed to execute 'send' on 'WebSocket': Still in CONNECTING state." *Stack Overflow*, 14 Apr. 2014, <[`stackoverflow.com/questions/23051416/`](https://stackoverflow.com/a/23052382)>.
* hexacyanide. "Rate-limiting the data sent down WebSockets." *Stack Overflow*, 13 Sept. 2013, <[`stackoverflow.com/questions/18782179/rate-limiting-the-data-sent-down-websockets/18820254#18820254`](https://stackoverflow.com/a/18820254)>.
* kasceled. "Redux+Websockets: Why manage this using middleware?" *Stack Overflow*, 25 Sept. 2017, <[`stackoverflow.com/questions/46402591/reduxwebsockets-why-manage-this-using-middleware`](https://stackoverflow.com/questions/46402591/reduxwebsockets-why-manage-this-using-middleware)>.
* Preble, Will. "TypeScript with React Functional Components." *DEV Community*, 16 July 2020, <[`dev.to/wpreble1/typescript-with-react-functional-components-4c69`](https://dev.to/wpreble1/typescript-with-react-functional-components-4c69)>.
* Retsam. "Can't get \`lodash.debounce()\` to work properly? Executed multiple times... (react, lodash, hooks)." *Stack Overflow*, 4 Dec. 2019, <[`stackoverflow.com/questions/59183495/cant-get-lodash-debounce-to-work-properly-executed-multiple-times-reac/59184678#59184678`](https://stackoverflow.com/a/59184678)>.
* Rippon, Carl. "Using Lodash Debounce with React and TypeScript." *Building SPAs - Carl Rippon*, 22 Dec. 2021, <[`carlrippon.com/using-lodash-debounce-with-react-and-ts`](https://www.carlrippon.com/using-lodash-debounce-with-react-and-ts/)>.
* syduki. "React + websocket: is there a way to throttle to limit the number of times the component gets updated by websocket." *Stack Overflow*, 13 Mar. 2021, <[`stackoverflow.com/questions/66615601/react-websocket-is-there-a-way-to-throttle-to-limit-the-number-of-times-the-c/66616016#66616016`](https://stackoverflow.com/a/66616016)>.
* "WebSocket Client." *WebSocket Test Page Client*, LivePerson, Inc., <[`livepersoninc.github.io/ws-test-page`](http://livepersoninc.github.io/ws-test-page/)>. Accessed 6 Jan. 2022.
