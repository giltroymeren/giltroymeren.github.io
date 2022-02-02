---
layout: post
title:  "Notes on React with Redux and Redux-Thunk: Part 2"
date:   2022-02-02
categories: notes javascript react redux typescript
---

This is a continuation of my previous post regarding the lessons learned from Stephen Grider's "*Master React and Redux with React Router, Webpack, and Create-React-App. Includes Hooks!*" UDemy course.

It will show a step-by-step application of the Redux theory for a better view of how the helpers the libraries provide are used in the actual React codebase.

## Table of Contents
* TOC
{:toc}

## Integrate Redux to the React codebase

> The file structure used in the following examples will mirror the output of `create-react-app` with TypeScript

In order to make the examples more concrete, the goal of the examples is to fetch data from [`https://jsonplaceholder.typicode.com`](https://jsonplaceholder.typicode.com).

### Prepare the `store` for the entire application

Add the following setup in `src/index.tsx` or the file where the root renderer is set.
   
1. Declare the `store`

    ~~~ javascript
    import { createStore } from 'redux'

    const store = createStore()
    ~~~

2. Add the reducer (not yet created)

    For context, the reducer here represents the combined reducers of the entire app.

    ~~~ javascript
    import { createStore } from 'redux'
    import reducers from './reducers'

    const store = createStore(reducers)
    ~~~

3. Add the `thunk` as a middleware

    Skip this if middlewares are not needed.

    ~~~ javascript
    import { createStore, applyMiddleware } from 'redux'
    import thunk from 'redux-thunk'
    import reducers from './reducers'

    const store = createStore(reducers, applyMiddleware(thunk))
    ~~~

4. Add the `store` as a provider to the React renderer

    ~~~ javascript
    import { createStore, applyMiddleware } from 'redux'
    import thunk from 'redux-thunk'
    import reducers from './reducers'

    const store = createStore(reducers, applyMiddleware(thunk))

    ReactDOM.render(
      <Provider store={store}>
        <App />
      </Provider>,
      document.getElementById('root')
    )
    ~~~

#### The `store` declaration can also be set in another file

The file is usually set in `src/store/index.ts` and import to the main file.

~~~ javascript
import { createStore, applyMiddleware } from 'redux'
import thunk from 'redux-thunk'
import { rootReducer } from '../reducers'

export const store = createStore(rootReducer, applyMiddleware(thunk))
~~~

~~~ javascript
import { store } from './store'

ReactDOM.render(
  <Provider store={store}>
  ...
~~~

### Prepare temporary reducers to fulfill `combineReducers`

Prepare the file `src/reducers/index.ts`. For now, the simple string-type return will suffice.

~~~ javascript
import { combineReducers } from "redux"

export default combineReducers({
  post: () => 'post'
})
~~~

### Create the base network request file

The `axios` library is used to perform network connections.

This is just a pattern used personally by Grider. I believe this is specific to `axios` to setup the base URL.

This is usually set in `api/<nameOfModel>.ts`.

~~~ javascript
import axios from 'axios'

export default axios.create({
  baseURL: `https://jsonplaceholder.typicode.com`
})
~~~

### Prepare the action creators with `thunk`

Katz presented two action creators: old and new. The latter is recommended because the first will return errors.

a. Old (incorrect) way

  ~~~ javascript
  export const fetchPosts = async () =>
      const response: IPost[] = await jsonPlaceholder.get('/posts')
      dispatch({
        type: "FETCH_POSTS",
        payload: response
      })
    }
  ~~~

b. New (correct) way

  ~~~ javascript
  export const fetchPosts = () =>
    async (dispatch: ThunkDispatch<{}, {}, AnyAction>): Promise<void> => {
      const response: IPost[] = await jsonPlaceholder.get('/posts')
      dispatch({
        type: "FETCH_POSTS",
        payload: response
      })
    }
  ~~~

Now we can put all network and asynchronous processes in this pattern!

#### Breakdown of the action creator

These are usually declared in `src/actions/index.ts`.

~~~ javascript
import { ThunkDispatch } from 'redux-thunk'
import { AnyAction } from 'redux'

export const fetchPosts = () =>
  async (dispatch: ThunkDispatch<{}, {}, AnyAction>): Promise<void> => {
    const response: IPost[] = await jsonPlaceholder.get('/posts')
    dispatch({
      type: "FETCH_POSTS",
      payload: response
    })
  }
~~~

The type of `dispatch` is from Visual Code's recommended type.

### Call the action creator in the component

#### Traditional way

Redux's `connect` is used here.

~~~ javascript
import { connect } from 'react-redux'
import React, { useEffect } from 'react'

interface IPostListProps {
  posts: IPost[]
  fetchPosts: () => Promise<void>
}

const PostList: React.FC<IPostListProps> = ({
  posts,
  fetchPosts
}) => {
  useEffect(() => {
    fetchPosts()
  }, [])

  return ( ... )
}

const mapStateToProps = ...

export default connect(
  mapStateToProps,
  { fetchPosts }
)(PostList);

~~~

#### Hooks way

`useDispatch` and `useEffect` will come in handy. These two are alternatives to using `mapDispatchToProps`.

~~~ javascript
function PostList() {
  const dispatch = useDispatch()

  useEffect(() => {
    dispatch(fetchPosts())
  }, [])
}
~~~

I haven't tried this much so use the former way for now!

### Create different files or **slices** for different models

Models refer to reducers in this context and will be combined for the `store`. Update the root reducer file:

~~~ javascript
export default combineReducers({
  posts: postReducer,
  users: userReducer,
  ...
})
~~~

The different reducer files can have the following structure:

- `src/reducers/index.ts`
- `src/reducers/postReducer.ts`
- `src/reducers/userReducer.ts`

### Final word on the state

Remeber that we have the following structure for our reducers in the store:

~~~ javascript
export default combineReducers({
  posts: postReducer,
  users: userReducer,
})
~~~

This means that the *single* `state` of our application will have the following corresponding structure depending on the interface type:

~~~ javascript
{
  posts: [{ ... }],
  users: [{ ... }]
}
~~~

#### Declare a root state

This root state will be used for several parts of the file.

##### In the root reducer file

~~~ javascript
export interface IRootState {
  posts: IPost[],
  users: IUser[]
}

export default combineReducers({
  posts: postReducer,
  users: userReducer
})
~~~

##### In the `store` file

This way is stated in the official documentation.

~~~ javascript
import { rootReducer } from '../reducers'

export const store = createStore(rootReducer, applyMiddleware(thunk))

export type TRootState = ReturnType<typeof rootReducer>
~~~

#### Use root state in components

- in a component's `mapStateToProps` arguments:

    ~~~ javascript
    const mapStateToProps = (state: TRootState) => {
      return {
        selectedVideo: state.videos.selected
      }
    }
    ~~~

### Notes

#### Root state and reducer state 

The Root state is not equal to a single reducer's state. The Root state is a combination of multiple reducers' own states!

Say we have the following reducer:

~~~ javascript
interface IState {
  list: IVideo[]
  selected: IVideo | null
}

export default (
  state: IState = { list: [], selected: null },
  action: IClearVideosActionType
    | IClearSelectedVideoActionType
    | IGetVideosActionType
    | ISelectVideoActionType
) => {
  switch (action.type) {
~~~

`IState` is set both as an interface and together with its initial state in order for a more robust TS declaration.

In component files, when this particular state is needed, it will work alongside the root state when its parts are needed.

## References

- Dawson, Sam. “useSelector vs connect (react-redux).” *Sam Dawson*, 1 Feb. 2021, [`samdawson.dev/article/react-redux-use-selector-vs-connect/#connect-function-testing`](https://www.samdawson.dev/article/react-redux-use-selector-vs-connect/#connect-function-testing).
- Grider, Stephen. “Modern React with Redux Training Course.” *Udemy*, uploaded by Stephen Grider, 1 Jan. 2022, [`udemy.com/course/react-redux`](https://www.udemy.com/course/react-redux).
- Pelissari, Eduardo Poça. “What is the main difference between using React-Redux Hooks and React-Redux Connect()?” *Stack Overflow*, 20 Sept. 2019, [`stackoverflow.com/questions/58027300/what-is-the-main-difference-between-using-react-redux-hooks-and-react-redux-conn/58027679#58027679`](https://stackoverflow.com/questions/58027300/what-is-the-main-difference-between-using-react-redux-hooks-and-react-redux-conn/58027679#58027679).
- “Usage with TypeScript \| React Redux.” *React Redux*, 30 May 2021, [`react-redux.js.org/using-react-redux/usage-with-typescript#define-root-state-and-dispatch-types`](https://react-redux.js.org/using-react-redux/usage-with-typescript#define-root-state-and-dispatch-types).