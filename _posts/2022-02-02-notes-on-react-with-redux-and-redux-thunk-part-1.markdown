---
layout: post
title:  "Notes on React with Redux and Redux-Thunk: Part 1"
date:   2022-02-02
categories: notes javascript react redux
---

The integration of Redux and Redux-Thunk has been a difficult experience for me in terms of better recall when building from scratch. I have read various documentation articles, blog posts and watched tutorials for the past two years and none stuck to me significantly to increase my confidence in handling this important feature in React application development.

Fortunately I recently finished Stephen Grider's "*Master React and Redux with React Router, Webpack, and Create-React-App. Includes Hooks!*" UDemy course and his sections on both Redux and Redux-Thunk made so much sense to me that I wrote this notes article for a digested reference for future me!

## Table of Contents
* TOC
{:toc}

## Redux and Redux-Thunk 

The diagrams below are adapted from Grider's presentation during the course.

> The file structure context mentioned in the following sections will mirror the output of `create-react-app`

### Redux Cycle

The integration of Redux to a React application starts with the `store` fed into a `Provider` that wraps over the root component. 

In this `store` object the reducers and other middleware (such as the `thunk`) are set as arguments which serves as the main contact point for the other ingredients of this process.

<br />

![Redux Cycle](/images/posts/2022-02-02-notes-on-react-with-redux-and-redux-thunk/redux-cycle.png){:class="img-responsive"}

<br />

The **action creator** (AC) defines the actual handling of whatever data in the state needs to be done. This has two kinds of returns: either a function or an object; this will be discussed later. Regardless, its final output will be an object with the following familiar structure:

~~~ javascript
const myAction = {
  type: "ACTION_TYPE",
  payload: "payload"
}
~~~

This is the **action**, a simple JS object with a type and an optional payload. Within the AC, there is also a `dispatch()` method provided to execute the action for the reducers to evaluate. 

#### Actions can be executed in various ways

1. Internally within the AC (through the `thunk` way):

    ~~~ javascript
    export const getVideos = (term: string) =>
      async (dispatch: TAppDispatch): Promise<void> => {
        const response: IYouTubeResults = await youtube.get('/search', {
          params: {
            q: term
          }
        })

        dispatch({
          type: EActionTypes.getVideos,
          payload: response.data.items
        })
      }  
    ~~~

2. Directly provided when the AC is called inside a component:

    `connect` and `mapDispatchToProps` work with this pattern.

    ~~~ javascript
    interface IVideoItem {
      video: IVideo
      selectVideo: (video: IVideo) => { type: EActionTypes, payload: IVideo }
    }

    const VideoItem: React.FC<IVideoItem> = ({
      video,
      selectVideo
    }) => {
      const onClick = () => {
        selectVideo(video)
      }

      return (
        ...
      )
      
    export default connect(null, { selectVideo })(VideoItem)
    ~~~~

3. Hooks way with `useDispatch`

Finally, the **reducers** directly receive the actions and perform changes to the **state** depending on their content. This automatically connect to the `store` object and will reflect to listening child components in the app.

This is how to use the Redux Cycle to perform changes to the app's overall state.

### Middlewares in Redux

Grider provided the following characteristics and remarks on Redux middlewares:

- a function that gets called with every action we dispatch.
- has the ability to *stop*, *modify* or otherwise mess around with actions.
- tons of open source middleware exist.
- most popular use of middlewares is for dealing with asynchronous actions.
- "Redux-Thunk" is a popular middleware for handling asynchronous issues.

#### Redux and middleware cycle

The inclusion of a middleware makes it easy to separate the responsibilities of different files in the project.

<br />

![Redux-Thunk Middleware Cycle](/images/posts/2022-02-02-notes-on-react-with-redux-and-redux-thunk/redux-thunk-middleware-cycle.png){:class="img-responsive"}

<br />

The code below is a simplified, compact version of an AC that wraps over an asynchronous function that dispatches an action within itself.

~~~ javascript
export const getVideos = (term: string) =>
  async (dispatch: TAppDispatch): Promise<void> => {
    const response: IYouTubeResults = await youtube.get('/search', {
      params: {
        q: term
      }
    })

    dispatch({
      type: EActionTypes.getVideos,
      payload: response.data.items
    })
  }  
~~~

### Redux-Thunk 

This was an eye-opener for me because I wanted a way to organize the async operations in separate files. Before, I was just stuck in declaring them inside the component itself leading to a stiff application.

Grider shared the following differences and similarities of action creators with and without Redux-Thunk:

<table>
<thead>
  <tr>
    <th></th>
    <th>Default Rules</th>
    <th>Rules with Redux-Thunk</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td rowspan="2">Return type</td>
    <td rowspan="2">ACs must return action objects</td>
    <td>ACs can return action objects</td>
  </tr>
  <tr>
    <td>ACs can return functions</td>
  </tr>
  <tr>
    <td>Action `type` property</td>
    <td>Actions must have a type property</td>
    <td>If an action object gets returned,<br>then it must have a type</td>
  </tr>
  <tr>
    <td>Action `payload` property</td>
    <td colspan="2">Actions can optionally have a payload</td>
  </tr>
</tbody>
</table>

It mentioned that there are two kinds of ACs in the Redux-Thunk context. These will be discussed below.

#### Two kinds of action creators in Redux-Thunk

The default Redux action creator returns an action object but in the addition of Redux-Thunk, it can also return a function.

Nevertheless, either way the function must be evaluated to an action object once it is dispatched to the reducers.

The addition of function capabilities enables a powerful way to handle certain processes such as asynchronous network connections as seen in the `getVideos()` method defined earlier.

<br />

![Redux-Thunk Cycle](/images/posts/2022-02-02-notes-on-react-with-redux-and-redux-thunk/redux-thunk-cycle.png){:class="img-responsive"}

<br />

## References

- Dawson, Sam. “useSelector vs connect (react-redux).” *Sam Dawson*, 1 Feb. 2021, [`samdawson.dev/article/react-redux-use-selector-vs-connect/#connect-function-testing`](https://www.samdawson.dev/article/react-redux-use-selector-vs-connect/#connect-function-testing).
- Grider, Stephen. “Modern React with Redux Training Course.” *Udemy*, uploaded by Stephen Grider, 1 Jan. 2022, [`udemy.com/course/react-redux`](https://www.udemy.com/course/react-redux).
- Pelissari, Eduardo Poça. “What is the main difference between using React-Redux Hooks and React-Redux Connect()?” *Stack Overflow*, 20 Sept. 2019, [`stackoverflow.com/questions/58027300/what-is-the-main-difference-between-using-react-redux-hooks-and-react-redux-conn/58027679#58027679`](https://stackoverflow.com/questions/58027300/what-is-the-main-difference-between-using-react-redux-hooks-and-react-redux-conn/58027679#58027679).
- “Usage with TypeScript \| React Redux.” *React Redux*, 30 May 2021, [`react-redux.js.org/using-react-redux/usage-with-typescript#define-root-state-and-dispatch-types`](https://react-redux.js.org/using-react-redux/usage-with-typescript#define-root-state-and-dispatch-types).