---
layout: post
title:  "HackerRank 2021 - Assessment Science - Front End React: Part 1"
date:   2021-12-31
categories: notes javascript react hackerrank
---

Yesterday I got a mail for a surprise React-related exam by HackerRank and of course I wanted to try it! 

The exam was composed of seven items with two coding challenges that needed to be solved under 90 minutes. I will discuss the items and how I solved each one.

## Table of Contents
* TOC
{:toc}

## React: Timer Component

### Problem

The challenge was to complete a small React application that shows an arbitrary integer as a decreasing timer with a stop button to halt execution.

The five definitions of done criteria are the following:

- The timer value decreases by 1 every second. For example, if the initial value is 100, after 1 second it becoms 99, etc.
- The value starts to decrease when the component is mounted.
- The initial value of the timer is set by a prop, *initial*.
- One the counter reachers 0, it should stop.
- The "Stop Timer" button stops the timer at its current value.

### Solution

I simply modified the `Timer` component and added the necessary state changes to manipulate the value and added an event listener for the stop button.

~~~ javascript
import React, { useEffect, useState } from "react";
import "./index.css";
import PropTypes from 'prop-types'

const Timer = ({ initial }) => {
  const [current, setCurrent] = useState(initial)

  let myTimer = null
  useEffect(() => {
    if(current === 0) return

    myTimer = setTimeout(() => setCurrent(current-1), 1000)
  })

  const stopTimer = () => clearTimeout(myTimer)

  return (
    <div className="mt-100 layout-column align-items-center justify-content-center" >
      <div className="timer-value" data-testid="timer-value">{current}</div>
      <button className="large" data-testid="stop-button" onClick={stopTimer}>Stop Timer</button>
    </div >
  )
}

Timer.propTypes = {
  initial: PropTypes.number.isRequired,
}

export default Timer
~~~

### Testing

The item has tests and unfortunately three out of four items fail. 

~~~ 
  ✓ initial UI is rendered as expected (8ms)
  ✕ inital value is set via props (4ms)
  ✕ timer stops at 0 (3ms)
  ✕ stop timer button stops the timer (3ms)
~~~

I still can't wrap my head around `jest.runTimersToTime()` and how I will make it work with the `useEffect` hook. 

The actual values of the timer received shows that only a second has passed after each execution of the timer mock even though I provided the 1000ms delay to tghe 

~~~
 FAIL  src/App.test.js
  ● inital value is set via props

    expect(element).toHaveTextContent()
    
    Expected element to have text content:
      110
    Received:
      119
      
      at Object.<anonymous> (src/App.test.js:32:38)
          at new Promise (<anonymous>)
      at node_modules/p-map/index.js:46:16
      at processTicksAndRejections (node:internal/process/task_queues:96:5)

  ● timer stops at 0

    expect(element).toHaveTextContent()
    
    Expected element to have text content:
      0
    Received:
      4
      
      at Object.<anonymous> (src/App.test.js:41:38)
          at new Promise (<anonymous>)
      at node_modules/p-map/index.js:46:16
      at processTicksAndRejections (node:internal/process/task_queues:96:5)

  ● stop timer button stops the timer

    expect(element).toHaveTextContent()
    
    Expected element to have text content:
      40
    Received:
      49
      
      at Object.<anonymous> (src/App.test.js:51:38)
          at new Promise (<anonymous>)
      at node_modules/p-map/index.js:46:16
      at processTicksAndRejections (node:internal/process/task_queues:96:5)
~~~

**I will update the code next time with the passing execution**. 

My solution is available here [`github.com/giltroymeren/react-timer`](https://github.com/giltroymeren/react-timer).
