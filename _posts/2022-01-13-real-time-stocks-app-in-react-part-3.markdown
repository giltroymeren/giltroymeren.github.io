---
layout: post
title:  "Real-time Stocks App in React: Part 3"
date:   2022-01-13
categories: notes series javascript typescript react react-hooks websocket
---

Our application is still not a "real-time stock app" as it name suggest so this post will implement the remaining features to display the stock prices in *real-time*.

For reference, the repository is available at [`github.com/giltroymeren/stocks-app-ts`](https://github.com/giltroymeren/stocks-app-ts).

## Table of Contents
* TOC
{:toc}

## Steps

### Create the `StockItem` component

The `stockList` state attribute is changed from an array of string ISINs to an array of `IStockItems` inside the `src/components/StockItem.tsx` component. In the meantime, the ISIN is only declared for now.

~~~ javascript
import React from 'react'

interface IStockItem {
  isin: string
}

const StockItem = (props: IStockItem) => {
  const { isin } = props

  return (
    <tr>
      <td>{isin}</td>
    </tr>
  )
}

export default StockItem
~~~

The `StockList` component is also updated with this new component:

~~~ javascript
const StockList = () => {
...
  return (
    <table>
      <tbody>
        {stockList.map((item: string, index: number) => {
          return <StockItem key={index} isin={item} />
        })}
      </tbody>
    </table>
~~~

### Subscribe and unsubscribe to the websocket

Now that a component and a type handle each ISIN, the next step is to mirror the latter's structure into what is actually returned by the API. But before that, we have to get these data from the server.

The API documentation states that the following objects are needed to start subscription or to terminate it.

~~~ json
// subscribe
{ "subscribe": <12-digit ISIN> }

// unsubscribe
{ "unsubscribe": <12-digit ISIN> }
~~~

The following are implementations for the two features inside `StockProvider`:

~~~ javascript
  const subscribeToServer = (isin: string) => {
    try {
      if (webSocket.current !== undefined) {
        if (webSocket.current.readyState !== WebSocket.OPEN) return;

        console.info('Subscribing via WS')
        webSocket.current.send(
          JSON.stringify({ "subscribe": isin })
        )
      }
    } catch (error: any) {
      console.error(error.message)
    }
  }

  const unsubscribeFromServer = (isin: string) => {
    try {
      if (webSocket.current !== undefined) {
        if (webSocket.current.readyState !== WebSocket.OPEN) return;

        console.info('Unsubscribing from WS')
        webSocket.current.send(
          JSON.stringify({ "unsubscribe": isin })
        )
      }
    } catch (error: any) {
      console.error(error.message)
    }
  }
~~~

The check for validity of `webSocket.current` is there to ensure it is accessible when the `readyState` attribute is read.

In the `StockItem` component, these two methods can be executed by the user through buttons after each ISIN:

~~~ javascript
const StockItem = (props: IStockItem) => {
  const { isin } = props

  const { subscribeToServer, unsubscribeFromServer } = useContext(StockContext)

  ...

  return (
    <tr>
      <td>{isin}</td>
      <td>
        <button onClick={() => subscribeToServer(isin)}>SUB</button>
      </td>
      <td>
        <button onClick={() => unsubscribeFromServer(isin)}>UNSUB</button>
      </td>
~~~

### Access the data received

Now that we can send which ISIN to subscribe to the server, the `onmessage` attribute of websocket should have access to its response:

~~~ javascript
webSocket.current.onmessage = function (event: any) {
  console.info('Receiving message')

  const data = JSON.parse(event.data)
  console.log(data)
  // TODO set data to UI
}
~~~

The data will be logged for now. The feature to display it to the UI is next!

### Store and display stock data from the server

First the `IStockItem` interface needs to be updated to reflect the API structure:

~~~ javascript
export interface IStockItem {
  isin: string,
  price: number,
  bid: number,
  ask: number
}
~~~

In turn the component should reflect these changes. `toFixed(n)` is just a native JS method to round the number (or string) to the nearest *n*.

~~~ javascript
const StockItem = ({ ...props }) => {
  const { isin, price, bid, ask } = props.stock

  const { subscribeToServer, unsubscribeFromServer } = useContext(StockContext)

  return (
    <tr>
      <td>{isin}</td>
      <td>{price.toFixed(2)}</td>
      <td>{bid.toFixed(2)}</td>
      <td>{ask.toFixed(2)}</td>
      <td>
        <button onClick={() => subscribeToServer(isin)}>SUB</button>
      </td>
      <td>
        <button onClick={() => unsubscribeFromServer(isin)}>UNSUB</button>
      </td>
    </tr>
  )
~~~

Upwards the parent `StockList`, we update the map renderer:

~~~ javascript
return (
  
  ...

        <tbody>
          {stockList.map(stock => {
            return <StockItem key={stock.isin} stock={stock} />
          })}
        </tbody>
~~~

Lastly, the `addStock` method in the actions should replace the string payload with the new default object:

~~~ javascript
const addStock = (isin: string) => {
  const newStock = {
    isin: isin,
    price: 0,
    bid: 0,
    ask: 0
  }
  dispatch({
    type: ACTION_ADD_STOCK,
    payload: [...state.stockList, newStock]
  })
~~~

At this point, the home page should show the table and list stocks but with empty amounts:

<br />

![Stocks App - Home Page with table of stocks with 0 amounts](/images/posts/2022-01-real-time-stocks-app-in-react/home-page-part-2-no-update.svg){:class="img-responsive"}

<br />

### Implement the `updateStock` action method

We need to fill those zeroes with actual amounts from the server!

The method is defined inside `StockProvider`:

~~~ javascript
const updateStockList = (data: IStockItem) => {
  dispatch({
    type: ACTION_UPDATE_STOCK_LIST,
    payload: data
  })
}
~~~

with a corresponding reducer to replace a stock in the `stockList` of the same unique ISIN.

~~~ javascript
case ACTION_UPDATE_STOCK_LIST:
  const data = action.payload
  const newList = state.stockList.map((stock: IStockItem) => {
    if (stock.isin === data.isin) {
      stock.price = data.price
      stock.bid = data.bid
      stock.ask = data.ask
    }
    return stock
  })

  return {
    ...state,
    stockList: newList
  }
~~~

Finally the websocket's `onmessage` attribute should be updated to pass the response data to this new method:

~~~ javascript
webSocket.current.onmessage = function (event: any) {
  const data = JSON.parse(event.data)
  updateStockList(JSON.parse(event.data))

  ...
~~~

This should show each ISIN's prices in *real-time* when their respective subscribe button is click and stop changes when the unsubscribe button is clicked.

<br />

![Stocks App - Home Page with table of stocks with 0 amounts](/images/posts/2022-01-real-time-stocks-app-in-react/home-page-part-2-subscribe.svg){:class="img-responsive"}

<br />

### Remove a stock from the table

Lastly, users should be able to remove a stock from the table. This is just a matter of removing the `IStockItem` object from the `stockList` array.

The action will accept an ISIN string from the UI.

~~~ javascript
const removeStock = (isin: string) => {
  dispatch({
    type: ACTION_REMOVE_STOCK,
    payload: isin
  })
}
~~~

Next, the reducer will filter out this object based from the ISIN.

~~~ javascript
case ACTION_REMOVE_STOCK:
  const removeList = state.stockList.filter((stock: IStockItem) =>
    stock.isin !== action.payload)

  return {
    ...state,
    stockList: removeList
  }
~~~

Completing the feature is to use the `removeStock()` method in the `StockItem` component:

~~~ javascript
const StockItem = ({ ...props }) => {
  const {
    subscribeToServer,
    unsubscribeFromServer,
    removeStock,
  } = useContext(StockContext)

  ...

  const onRemove = () => {
    unsubscribeFromServer(isin)
    removeStock(isin)
  }

  ...
  
  return (

    ...
          <td>
            <button onClick={onRemove}>REMOVE</button>
          </td>
~~~

Clicking the remove button will not only delete the item from the state but will unsubscribe it first to make sure it is no longer subscribed during the process.

## Next steps

We now have a working, basic stocks app. During the development I observed the fast receipt of numerous responses from the server and I wanted to add a feature to control this.

The final part of this series will deal with such problems that is commonly implemented in industry to handle large volumes of asynchronous data.

## References
* Gigablah. "Uncaught InvalidStateError: Failed to execute 'send' on 'WebSocket': Still in CONNECTING state." *Stack Overflow*, 14 Apr. 2014, <[`stackoverflow.com/questions/23051416/`](https://stackoverflow.com/a/23052382)>.
* kasceled. "Redux+Websockets: Why manage this using middleware?" *Stack Overflow*, 25 Sept. 2017, <[`stackoverflow.com/questions/46402591/reduxwebsockets-why-manage-this-using-middleware`](https://stackoverflow.com/questions/46402591/reduxwebsockets-why-manage-this-using-middleware)>.
* Preble, Will. "TypeScript with React Functional Components." *DEV Community*, 16 July 2020, <[`dev.to/wpreble1/typescript-with-react-functional-components-4c69`](https://dev.to/wpreble1/typescript-with-react-functional-components-4c69)>.
* "WebSocket Client." *WebSocket Test Page Client*, LivePerson, Inc., <[`livepersoninc.github.io/ws-test-page`](http://livepersoninc.github.io/ws-test-page/)>. Accessed 6 Jan. 2022.
