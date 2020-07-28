---
layout: post
title:  "Notes on declaring React components"
date:   2020-07-10
categories: notes series javascript react
---

When I first started with React a few years ago through tutorials online and tech talks in my company, one of the things that confused me the most is the number of ways components were made.

Luckily there was a structured introduction by Cory House on the different kinds of components. The following are some of the observations I have made from using each in my projects.

## Table of Contents
* TOC
{:toc}

## Ways to declare

The are two super categories in declaring React components: class and function. This distinction roots from the fact that before ES2015 there were no native class implementation.

~~~ javascript
function Person(firstName, lastName) {
    this.firstName = firstName;
    this.lastName = lastName;
}

var Nurse = new Person('Juan', 'dela Cruz');
~~~

> I included simple state management and click handlers in all the examples as a additional overview on each way's syntax.

### class components

#### `React.createClass`

This is available since the first edition of React.

~~~ javascript
var LikeButton = React.createClass({
    getInitialState: function() {
        return {
            checked: false
        };
    },

    handleClick: function() {
        this.setState(function previousState() {
            return { checked: !previousState.checked };
        });
    },

    render: function() {
        return (
            <button onClick={ this.handleClick }>
                Like
            </button>
        );
    }
});
~~~

<br />

|                       | Remarks   |
|---------------------- |---------  |
| Constructor           | `getInitialState` |
| State initialization  | Inside `getInitialState` and return an object of attributes |
| State modification    | `this.setState()` |
| Props usage           | `this.props.propName` |
| Click handlers        | autobinded |
| Render                | See code |


#### regular class

These ES2015 class components extends the `React.Component` base class.

~~~ javascript
class LikeButton extends React.Component {
    constructor(props) {
        super(props);
        this.state = { checked: false };

        this.handleClick = this.handleClick.bind(this);
    }

    handleClick() {
        this.setState(previousState => (
            { checked: !previousState.checked }
        ));
    }

    render() {
        return (
            <button onClick={ this.handleClick }>
                Like
            </button>
        );
    }
}
~~~

<br />

|                       | Remarks   |
|---------------------- |---------  |
| Constructor           | See code |
| State initialization  | `this.state` inside the constructor |
| State modification    | `this.setState()` |
| Props usage           | Passed into the constructor |
| Click handlers        | Requires binding in the constructor |
| Render                | See code |

##### Property initialization

We know that arrow functions are autobinded inside classes. With this in mind, we can attempt to convert `handleClick()` into an arrow function but just keep in mind the consistency of your component methods.

~~~ javascript
class Like extends React.Component {
    state: {
        checked: false
    }

    handleClick = () => {
        this.setState(previousState => (
            { checked: !previousState.checked }
        ));
    }

    render = () => {
        ...
~~~

- `constructor` is removed because we can initialize the state outside it.
-  Regarding application to other methods Tanudjaja states that "(You) can even use the property initializers syntax for render, and all other component lifecycle methods if you like.".

> Keep in mind that function hoisting is only applicable to non-arrow functions. Beware of the organization of the property initializer methods.

### Functional components

These components are declared as JS functions and are usually called "stateless" before React 16.8 because they do not have access to the set of lifecycle methods such as `componentWillMount` and others.


#### regular function

These are ES2015 function components.

~~~ javascript
function Like(props) {
    var [checked, setChecked] = React.useState(false);

    function handleClick() {
        setChecked(!checked);
    }

    return (
        <button onClick={ handleClick }>
            Like
        </button>
    );
}
~~~

<br />

|                       | Remarks   |
|---------------------- |---------  |
| Constructor           | Use hooks |
| State initialization  | `React.useState` |
| State modification    | `React.useState` |
| Props usage           | `props.propName` |
| Click handlers        | See code |
| Render                | A simple `return` statement |


I assumed that `handleClick()` declared as a regular function will not work because in the class components it needs to be binded in constructors unless declared as an arrow function. I would like to report that it works in this situation.

#### Arrow function

~~~ javascript
function Like(props) {
    var [checked, setChecked] = React.useState(false);

    const handleClick = () => setChecked(!checked);

    return (
        <button onClick={ handleClick }>
            Like
        </button>
    );
}
~~~

<br />

|                       | Remarks   |
|---------------------- |---------  |
| Constructor           | Use hooks |
| State initialization  | `React.useState` |
| State modification    | `React.useState` |
| Props usage           | `props.propName` |
| Click handlers        | See code |
| Render                | A simple `return` statement |


## References
* Accomazzo, Anthony. “React.createClass vs. ES6 Class Components.” *Fullstack React*, Fullstack.io, LLC, 14 Feb. 2017, [`newline.co/fullstack-react/articles/react-create-class-vs-es6-class-components/`](www.newline.co/fullstack-react/articles/react-create-class-vs-es6-class-components/).
* Accomazzo, Anthony. “Use Property Initializers for Cleaner React Components.” *Fullstack React*, Fullstack.io, 21 Feb. 2017, [`newline.co/fullstack-react/articles/use-property-initializers-for-cleaner-react-components/`](www.newline.co/fullstack-react/articles/use-property-initializers-for-cleaner-react-components/).
* Ceddia, Dave. “Convert React.createClass to ES6 Class.” *Dave Ceddia*, Dave Ceddia, 11 May 2017, [`daveceddia.com/convert-createclass-to-es6-class/`](www.daveceddia.com/convert-createclass-to-es6-class/).
* “Components and Props.” *React*, Facebook, 2020, [`reactjs.org/docs/components-and-props.html`](reactjs.org/docs/components-and-props.html).
* Greene, Eric. “JavaScript ES2015 Classes and Prototype Inheritance (Part 1 of 2).” *Accelebrate*, Accelebrate, 27 Jan. 2016, [`accelebrate.com/blog/javascript-es6-classes-and-prototype-inheritance-part-1-of-2/`](www.accelebrate.com/blog/javascript-es6-classes-and-prototype-inheritance-part-1-of-2/).
* King, James. “React OnClick Event Handling (With Examples).” *Upmostly*, 29 Jan. 2020, [`upmostly.com/tutorials/react-onclick-event-handling-with-examples`](www.upmostly.com/tutorials/react-onclick-event-handling-with-examples).
* Kirszenberg, Alexandre. “What Is the Difference between Using Constructor vs GetInitialState in React / React Native?” *Stack Overflow*, 5 June 2015, [`stackoverflow.com/a/30668609`](www.stackoverflow.com/a/30668609).
* “React Without ES6.” *React*, Facebook, 2020, [`reactjs.org/docs/react-without-es6.html`](https://reactjs.org/docs/react-without-es6.html).
* Tanudjaja, Audy. “Property Initializers: What, Why, and How to Use It.” *Medium*, ITNEXT, 12 Feb. 2018, [`itnext.io/property-initializers-what-why-and-how-to-use-it-5615210474a3`](itnext.io/property-initializers-what-why-and-how-to-use-it-5615210474a3).
* Wieruch, Robin. “React Function Components.” *rwieruch*, 18 Mar. 2019, [`robinwieruch.de/react-function-component`](www.robinwieruch.de/react-function-component).
