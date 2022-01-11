---
layout: post
title:  "Real-time Stocks App in React: Part 1"
date:   2022-01-11
categories: notes series javascript typescript react react-hooks websocket
---

I developed a small React application that shows real time price details of a stock given their "International Securities Identification Numbering" number or simply ISIN.

This will be a series covering how I built this app and discuss the technologies and techniques used to implement each feature.

For reference, the repository is available at [`github.com/giltroymeren/stocks-app-ts`](https://github.com/giltroymeren/stocks-app-ts).

## Table of Contents
* TOC
{:toc}

## Introduction

My motivation for this app is [this efficient and fast WebSocket test page client](http://livepersoninc.github.io/ws-test-page/) by LivePerson, Inc. built entirely in native JavaScript! I wanted to do a version for stock lookup in ReactJS.

The application has three main parts: the search bar, the stock table and a connection button.

<br />

1. Empty stock table

    ![Stocks App - React component architecture](/images/posts/2022-01-real-time-stocks-app-in-react/home-page-empty.svg){:class="img-responsive"}

2. Filled stock table with prices

    ![Stocks App - React component architecture](/images/posts/2022-01-real-time-stocks-app-in-react/home-page.svg){:class="img-responsive"}

<br />

The search field adds an ISIN to the table. This table displays the ISIN's current price, bid and ask values together with buttons to update, stop update and remove each. Lastly the connect/discomment button is to keep track of websocket's status.

## Architecture

The current design of the app was mostly influenced by my trial and errors in making the websocket updates work with the UI.

<br />

![Stocks App - React component architecture](/images/posts/2022-01-real-time-stocks-app-in-react/react-component-diagram.svg){:class="img-responsive"}

<br />

Each stock is displayed as a `Stockitem` and are rendered from the `StockList` component. A stock is added through the `SearchBar` and the entire state is managed by the `StockProvider`. The `Footer` is just for presentational separation.

## Steps

### Setup the base

As with all React projects, we start with the reliable `create-react-app` command to generate a base build. I decided to use TypeScript to familiarize myself with it more.

~~~ node
npx create-react-app my-app --template typescript
~~~

I also updated the `public/index.html` file and wiped out several JS and CSS files whose content will not be in use in relation to this project.

### Create the `SearchBar` component

It is a simple component that will (eventually) add new stocks to the table.

The following is the content of the `src/components/SearchBar.tsx` file.

~~~ javascript
import React, { useContext, useState } from 'react'

const SearchBar = () => {
  const [isin, setISIN] = useState('')

  const onChange = (event: any) => {
    setISIN(event.target.value)
  }

  const onClick = (event: any) => {
    event.preventDefault()
    // TODO: add ISIN to state
    setISIN('')
  }

  return (
    <form>
      <input type="text"
        name="isin"
        placeholder='Enter the ISIN'
        value={isin}
        onChange={onChange} />
      <button onClick={onClick}>Check</button>
    </form>
  )
}

export default SearchBar
~~~

The component manages its own ISIN content of the text field. The next sections will discuss how to add this to the overall app state.

### Add a stock to the state

In order to add new ISINs, we need a way to store these in a centralized management. 

#### How to implement the context & provider

I decided to go to the provider way:

> To quote Pavlutin, "Using the context in React requires 3 simple steps: *creating* the context, *providing* the context, and *consuming* the context".

1. Create the context and provider

    The `createContext` method is used to initialize this step in `src/context/StockContext.tsx`.

    ~~~ javascript
    import React, { createContext, useReducer } from "react"
    import StockReducer, { ACTION_ADD_STOCK } from "./StockReducer"

    const StockContext = createContext()

    export const StockProvider: React.FC = ({ children }) => {
      const initialState = { stockList: [] }
      const [state, dispatch] = useReducer(StockReducer, initialState)

      const addStock = (isin: string) => {
        dispatch({
          type: ACTION_ADD_STOCK,
          payload: [...state.stockList, isin]
        })
      }

      return <StockContext.Provider
        value={{
          stockList: state.stockList,

          addStock
        }}>
        {children}
      </StockContext.Provider>
    }

    export default StockContext
    ~~~

    For state management, the `useReducer` hook is used to get a copy of the *current* state and a dispatch object to map reducers and actions together.

2. Define actions

    The `addStock` method was defined inside the provider. It will simply add the new isin to the `stockList` attribute of the state. This attribute is currently defined as an array of strings but this will be changed to a more formal ISIN object later.

3. Implement the reducers

    My personal format for naming action types is to add the prefix "ACTION_" to distinguish their purpose from other constants.

    In `src/context/StockReducer.ts` we simply return the state and assign a new value to the `stockList` attribute as defined in the actions.

    ~~~ javascript
    export const ACTION_ADD_STOCK = 'ADD_STOCK'

    const StockReducer = (state: any, action: any) => {
      switch (action.type) {
        case ACTION_ADD_STOCK:
          return {
            ...state,
            stockList: action.payload
          }
        default: return state
      }
    }

    export default StockReducer
    ~~~

4. Wrap the parent component (usually `<App />` itself) with the provider

    Finally we bring the provider to the main component `App`:

    ~~~ javascript
    import './App.css';
    import SearchBar from './components/SearchBar'
    import { StockProvider } from './context/StockContext'

    function App() {
      return (
        <StockProvider>
          <h1>Stocks App</h1>

          <SearchBar />
        </StockProvider>
      );
    }
    ~~~

    All the children will now have access to this central provider, as defined in its `value` attribute in `StockProvider.tsx`.

#### How to use provider in components

We can now call `addStock` inside the `SearchBar` component once the form is submitted.

The `useContext` hook will return a useable copy of a context provided. In turn, the values it contains will now be accessible either by destructuring or dot-access in the component calling it.

1. Destructuring:

    ~~~ javascript
      const { addStock } = useContext(StockContext)

      const onClick = (event: any) => {
        event.preventDefault()
        addStock(isin)
        setISIN('')
      }
    ~~~

2. Dot-access
  
    ~~~ javascript
      const stockContext = useContext(StockContext)

      const onClick = (event: any) => {
        event.preventDefault()
        stockContext.addStock(isin)
        setISIN('')
      }
    ~~~

I prefer the destructuring way because we eliminate a lot of repeated use of the `stockContext` variable name and simplifies the recall to the actual action method name.

When a use clicks the submit button, the `onClick` method will now add the new ISIN string to the state `stockList` array.

### Show the stock table

This is simply mapping over the `stockList` array and showing each ISIN.

The `src/components/StockList.tsx` also uses the `StockContext` to access the state.

~~~ javascript
import React, { useContext } from 'react'
import StockContext from '../context/StockContext'

const StockList = () => {
  const { stockList } = useContext(StockContext)

  if (stockList.length === 0) return <h3>Nothing to show...</h3>

  return (
    <table>
      <tbody>
        {stockList.map((item: string, index: number) => {
          return <tr key={index}><td>{item}</td></tr>
        })}
      </tbody>
    </table>
  )
}

export default StockList
~~~

I added a catch if the list is empty then we just show a message; otherwise the table is rendered. The ISIN is shown for now but this will include the amounts from the API later.

## Next steps

The next part will discuss the `WebSocket` integration with the React providers and components. This is the bulk of my new learnings with this solution so prepare for a LOT of explanations and realizations!

## References
* Pavlutin, Dmitri. "A Guide to React Context and useContext() Hook." *Dmitri Pavlutin Blog*, 6 Sept. 2021, <[`dmitripavlutin.com/react-context-and-usecontext`](https://dmitripavlutin.com/react-context-and-usecontext/)>.
* Pavlutin, Dmitri. "An Easy Guide to React useReducer() Hook." *Dmitri Pavlutin Blog*, 15 Sept. 2021, <[`dmitripavlutin.com/react-usereducer`](https://dmitripavlutin.com/react-usereducer/)>.
* "WebSocket Client." *WebSocket Test Page Client*, LivePerson, Inc., <[`livepersoninc.github.io/ws-test-page`](http://livepersoninc.github.io/ws-test-page/)>. Accessed 6 Jan. 2022.
  