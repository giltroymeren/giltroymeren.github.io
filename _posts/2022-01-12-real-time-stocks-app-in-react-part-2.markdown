---
layout: post
title:  "Real-time Stocks App in React: Part 2"
date:   2022-01-12
categories: notes series javascript typescript react react-hooks websocket
---

This second part continues the discussion from the previous post on how I built this real-time stocks application with React. 

For reference, the repository is available at [`github.com/giltroymeren/stocks-app-ts`](https://github.com/giltroymeren/stocks-app-ts).

## Table of Contents
* TOC
{:toc}

## Introduction

The skeleton components for adding a stock and displaying them in the UI are available. 

<br />

![Stocks App - Home Page Part 1](/images/posts/2022-01-real-time-stocks-app-in-react/home-page-part-1.svg){:class="img-responsive"}

<br />

This part will discuss how to use the `WebSocket` interface in connecting to an API that returns amount values for a given ISIN through subscription.

## Architecture

During the development of this app, the websocket methods were declared inside the `StockList` component in order to take advantage of the access to the child stocks.

However I wanted these asynchronous method definitions to be separate from the component files and near the state management code.

<br />

![Stocks App - StockProvider architecture](/images/posts/2022-01-real-time-stocks-app-in-react/stockprovider-diagram.svg){:class="img-responsive"}

<br />

## Steps

### Connect using the `WebSocket` interface

A `WebSocket` must be created once throughout the lifetime of the application (in this small app). 

Inside `StockProvider` file I declared the `webSocket` constant using the `useRef` hook which guarantees that this is the only instance we use.

~~~ javascript
const webSocket = useRef<any>()
~~~

Pavlutin stated the following rules about this hook:

> 1. The value of the reference is persisted (stays the same) between component re-renderings
> 2. Updating a reference doesn't trigger a component re-rendering

This is perfect because we don't want the instance to lose *that* initial connection from the server after component renderings (in this situation).

#### Define the `WebSocket`

Next I created the actual `connectedToServer()` method that defines custom handling for each feature.

~~~ javascript
export const StockProvider: React.FC = ({ children }) => {

...

  const connectToServer = () => {
    try {
      if (webSocket.current !== undefined && webSocket.current !== null) {
        return;
      }

      webSocket.current = new WebSocket(DEFAULT_URL);
      webSocket.current.onopen = function () {
        console.info('Connected to WS')
        // TODO set state to connected
      }
      webSocket.current.onmessage = function (event: any) {
        console.info('Receiving message')
        // TODO parse data from API
      }
      webSocket.current.onclose = function () {
        console.info('Disconnected from WS; attempting reconnection in 1 second... ')
        // TODO set state to disconnected

        setTimeout(() => connectToServer(), 1000)
      }
    } catch (event: any) {
      console.error(event.message)
    }
  }

...
~~~

1. An early exit check to prevent recreation of a new `WebSocket` instance if there's already one available
   
    Observe that the instance used is assigned to `webSocket.current` instead of the `webSocket` object instead. This is because the latter is a `const` and this is a common pattern with the `useRef` hook when we need a mutable attribute of the referenced instance.

2. The `webSocket.current` is assigned to a new `WebSocket` instance given an API URL

3. Three of the common features are defined: 

    - `onopen`: just logs that the connection was established successfully.
    - `onmessage`: this method receives data from the API; for now it also logs what happened but later the data will be used to update the state.

        The response for whatever is returned by the subscribe/unsubsribe calls will also be returned here.

    - `onclose`: I wanted the connection to be persistent so in case a disconnection happens, the `setTimeout` will reattempt to create a new one after a second.

Finally add the new method declared inside `StockProvider` to the values it provides:

~~~ javascript
...

  return <StockContext.Provider
    value={{
      stockList: state.stockList,

      addStock
      connectToServer,
    }}>
~~~

All other state attributes and any other actions or methods declared inside this provider will be accessible to the children.

#### Connect to the component

In order to execute `connectToServer()`, I put it inside the a `useEffect` in `StockList`.

~~~ javascript
const StockList = () => {
  const { stockList, connectToServer } = useContext(StockContext)

  useEffect(() => {
    connectToServer()
  })

...
~~~

#### Define context's initial state

Before we move on, I added a type for the `createContext` to establish the dependencies for the upcoming steps.

~~~ javascript
export interface IStockContext {
  stockList: string[],
  connectToServer: any,
  addStock: any,
}

const initialState = {
  stockList: [],
  connectToServer: () => {},
  addStock: () => { },
}

const StockContext = createContext<IStockContext>(initialState)
~~~

This is pretty complex now because three parts of this file should have the same contents: `IStockContext`, `initialState` and the value object in `<StockContext.Provider>`. Make sure to take note of this!

### Add `disconnectFromServer()`

Since we can now connect to the server, the disconnect should be easy to implement inside `StockProvider`.

~~~ javascript
// @ts-ignore
const disconnectFromServer = () => webSocket.current.close()
~~~

I added the `@ts-ignore` flag to prevent errors in checking `webSocket.current`'s value.

This method should be exported for use by the children in the values section.

### Add the connected state

In order to aid decisions in the UI regarding the websocket status, we can add reducers and actions for the `isConnected` status in the state.

~~~ javascript
const setConnected = () => dispatch({ type: ACTION_SET_CONNECTED })
const setDisconnected = () => dispatch({ type: ACTION_SET_DISCONNECTED })
~~~

These are called inside the `onopen` and `onclose` attributes of `connectToServer()`.

Their respective counterparts in `StockReducer.ts` will just set the `isConnected` state.

~~~ javascript
case ACTION_SET_CONNECTED:
  return {
    ...state,
    isConnected: true
  }

case ACTION_SET_DISCONNECTED:
  return {
    ...state,
    isConnected: false
  }
~~~

### Apply connection features to UI

As a fun feature I added a button on the UI to allow users to disconnect the app from the server and see the immediate persisting connection fallback.

<br />

![Stocks App - Home Page Part 1 with Disconnect Button](/images/posts/2022-01-real-time-stocks-app-in-react/home-page-part-1-disconnected.svg){:class="img-responsive"}

<br />

We access the `isConnected` state and `disconnectFromServer()` method through the context in the `StockList` component.

~~~ javascript
const StockList = () => {
  const {
    stockList,
    isConnected,
    connectToServer,
    disconnectFromServer
  } = useContext(StockContext)

...

  return (
    <>
      ...
      {isConnected ? <button onClick={() => disconnectFromServer()}>Disconnect</button> :
        <button onClick={() => connectToServer()}>Connect</button>}
~~~

Now we can connect/disconnect to the server!

## Next steps

Now that the technical side of the `WebSocket` integration with the React context is done, another part is in the following post. Each ISIN will be shown with real time data from the server.

## References
* Dmitri Pavlutin. “The Complete Guide to useRef() and Refs in React.” * Blog*, Dmitri Pavlutin, 26 Oct. 2021, <[`dmitripavlutin.com/react-useref-guide`](https://dmitripavlutin.com/react-useref-guide/)>.
* Kolman, Daniel. "Websocket is undefined immediately after receiving message in react." *Stack Overflow*, Stack Overflow, 20 Sept. 2021, <[`stackoverflow.com/questions/69249467/websocket-is-undefined-immediately-after-receiving-message-in-react/69294168#69294168`](https://stackoverflow.com/a/69294168)>.
* Gigablah. "Uncaught InvalidStateError: Failed to execute 'send' on 'WebSocket': Still in CONNECTING state." *Stack Overflow*, 14 Apr. 2014, <[`stackoverflow.com/questions/23051416/`](https://stackoverflow.com/a/23052382)>.
* Preble, Will. "TypeScript with React Functional Components." *DEV Community*, 16 July 2020, <[`dev.to/wpreble1/typescript-with-react-functional-components-4c69`](https://dev.to/wpreble1/typescript-with-react-functional-components-4c69)>.
* Tleukabiluly, Medet. "Websocket is undefined immediately after receiving message in react." *Stack Overflow*, Stack Overflow, 20 Sept. 2021, <[`stackoverflow.com/questions/69249467/websocket-is-undefined-immediately-after-receiving-message-in-react/69294168#69281974`](https://stackoverflow.com/a/69281974)>.
* "WebSocket - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 15 Dec. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/WebSocket`](https://developer.mozilla.org/en-US/docs/Web/API/WebSocket)>.
* "WebSocket Client." *WebSocket Test Page Client*, LivePerson, Inc., <[`livepersoninc.github.io/ws-test-page`](http://livepersoninc.github.io/ws-test-page/)>. Accessed 6 Jan. 2022.
  