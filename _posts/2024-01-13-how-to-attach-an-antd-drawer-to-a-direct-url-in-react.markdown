---
layout: post
title: "How to attach an antd drawer to a direct URL in React"
date: 2024-01-13
categories: notes javascript typescript react react-hooks react-routes antd
---

Last year, I had the chance to use [antd](https://ant.design) design library and got acquainted a lot with their Drawer component. Developed by the Ant Group (mainly Alibaba and Alipay), _antd_ is currently one of the leading UI libraries use with web frontend frameworks such as React.

The feature I was developing involved CRUD of tabulated data. During the design stage I decided with the UI/UX designer to allow users to also show the drawer if the access from the URL, at least the creation form, together with the traditional trigger from a button element.

## Table of Contents

- [Table of Contents](#table-of-contents)
- [Challenge](#challenge)
  - [Solution](#solution)
    - [Show a drawer from click](#show-a-drawer-from-click)
    - [Pair a route with the drawer](#pair-a-route-with-the-drawer)
    - [Open the drawer on URL visit](#open-the-drawer-on-url-visit)
      - [Check and compare the current URL](#check-and-compare-the-current-url)
      - [useNavigate](#usenavigate)
      - ["Guard" the drawer URL](#guard-the-drawer-url)
- [What's next?](#whats-next)
  - [What if we want more drawers?](#what-if-we-want-more-drawers)
- [References](#references)

## Challenge

My company's web application uses [react-router-dom](https://reactrouter.com/en/main) to manage all the routes. This is straightforward to learn if you have experience in structuring routes in a web app or simply through the provided tutorial in the official website.

On a nutshell, a simple setup directly in the `render` method with the elements `BrowserRouter`, `Routes` and `Route` can look like the following depending on the structure of your project:

~~~javascript
ReactDOM.createRoot(
  document.getElementById("root") as Element)
  .render(
    <React.StrictMode>
      <BrowserRouter>
        <Routes>
          <Route path="/" element={<PageLayout />}>
            <Route index path="/" element={<Home />} />
            <Route path="/about" element={<About />} />
            <Route path="/contact" element={<Contact />} />

            // ... other routes

            <Route path="*" element={<NoMatch />} />
          </Route>
        </Routes>
      </BrowserRouter>
    </React.StrictMode>
  );
~~~

For this example, I decided to put all other components under `PageLayout` for consistency. In this setup, users can visit any of the defined paths in the `path` attribute and their respective component should show up.

Now, how do we tie in this routing with drawers?  

### Solution

Each section is an overview of the steps I've tried to implement the requirement in a sandbox environment so it can be replicated in other projects.

#### Show a drawer from click

We add the "Basic drawer" example from the official documentation in the `Home` component. `useState` to take note of the drawer's status and provide a callback function to toggle its visibility.

~~~javascript
const Home: React.FC = () => {
  const [isDrawerOpen, setIsDrawerOpen] = React.useState(false);

  const toggleDrawer = () => setIsDrawerOpen(!isDrawerOpen);

  return (
    <>
      <h1>Home</h1>
      <Button type="primary" onClick={showDrawer}>
        Open Drawer
      </Button>

      <Drawer title="Basic Drawer" onClose={setIsDrawerOpen} open={isDrawerOpen}>
        <p>Lorem ipsum dolor sit amet...</p>
      </Drawer>
    </>
  );
};
~~~

The drawer should appear from the right side of the screen when the "Open Drawer" button is clicked.

#### Pair a route with the drawer

Now that we have a working drawer triggered from a button, we now want to attach it to a particular path, say "/details". Since the drawer resides in the `Home` component and in turn its route, we need to tell `react-router-dom` about this relationship as well.

It just takes a small modification in the routes set earlier:

~~~javascript
ReactDOM.createRoot(
  ...
      <BrowserRouter>
        <Routes>
          <Route path="/" element={<PageLayout />}>
          <Route path="/" element={<Home />}>
            <Route path="/details" element={<Details />} />
          </Route>
          <Route path="/about" element={<About />} />
          ...
~~~

Essentially the route of the drawer, now inside the `Details` component, is set as a child of the `Home` component. We remove the `index` attribute to enable this parent-child setting.

#### Open the drawer on URL visit

Next we will solve how to we let React now to change the current page URL to `/details` when we either click the "Open Drawer" button or directly go to URL.

Back in the `Home` component we will use the magic of several tools to manipulate the URL.

##### Check and compare the current URL

`react-router-dom` has a useful hook called `useLocation` the provides the current browser URL of the user as the variable `pathname`.

We can use this _current_ value to compare with an _incoming_ command to go to another URL which is, in this case, opening or showing the drawer.

~~~javascript
import { useLocation } from "react-router-dom";

const Home: React.FC = () => {
  const { pathname } = useLocation();
  const previousPath = usePreviousPath(pathname);
  ...
~~~

The hook `usePreviousPath` is a nifty tool that we can use, as it name suggests, to get the previous path for the comparison. The following is a TypeScript version of the code shared by Shubham Khatri that takes advantage of the `useRef` hook to take note of the current path that will be checked later when the appearance of the drawer is triggered hence this value will then be the previous path.

~~~javascript
const usePreviousPath = <T = any>(value: T) => {
  const ref = React.useRef<T | null>(null);

  React.useEffect(() => {
    ref.current = value;
  }, [value]);

  return ref.current;
};
~~~

##### useNavigate

The last hook of the day is the ever-useful `useNavigate` that provides the `navigate` callback function that takes in any path string and tells the router to go to that value in the browser.

##### "Guard" the drawer URL

This is the most important part of this solution. Basically in the `Home` component, we will always monitor any changes of the `pathname` value, check if its the drawer's and then act accordingly.

~~~javascript
const Home: React.FC = () => {
  ...
  React.useEffect(() => {
    if(pathname !== previousPath) {
      if (matchPath("/details", pathname) !== null) {
        toggleDrawer();
        return navigate("/details");
      }

      setIsDrawerOpen(false);
      return navigate("/");
    }
  }, [pathname])
~~~

The code checks if there were any changes in the value of `pathname` and if it's not the same as the previous value, it means there must be a `navigate` that was fired.

Next `matchPath` is used to confirm the actual URL value and if it matches the drawer URL, we show the drawer and point the URL to the same one. Otherwise we just keep the drawer closed and return to the component's route.

That's the building blocks of this solution. This is really helpful when we want users to skip the button click in certain cases such as coming from tutorials or instruction redirections from another part of the web app.

## What's next?

Now that we know how to solve this particular problem, there are other use cases that can be explored further such as connecting other toggleable UI elements such as modals (but I know modals are used for providing information only) or what if the drawers need to be futher down paths beyond the index page such as `/products/create` and `/products/edit` for CRUD cases?

Here I have one that takes advantage of `switch` cases and the `matchPath` function.

### What if we want more drawers?

There are cases when one may need to show multiple drawers in one page such as a complex form or just for the sake of information organisation.

For my solution, I add another state for identifying each drawer's contents. For example, we add `/details-2` drawer in the `Home` component:

~~~javascript
const Home: React.FC = () => {
  ...
  const [isDrawerOpen, setIsDrawerOpen] = React.useState(false);
  const [drawerPath, setDrawerPath] = React.useState("/details");

  React.useEffect(() => {
    if(pathname !== previousPath) {
      switch(true) {
        case matchPath("/details", pathname) !== null:
          toggleDrawer();
          setDrawerPath("/details");
          return navigate("/details");
        case matchPath("/details-2", pathname) !== null:
          toggleDrawer();
          setDrawerPath("/details-2");
          return navigate("/details-2");
        }
    ...
~~~

Then we set the buttons to trigger each URL.

~~~javascript
<Button type="primary" onClick={() => navigate("/details")}>
  Open Details # 1
</Button>
...
<Button type="primary" onClick={() => navigate("/details-2")}>
  Open Details # 2
</Button>
~~~

Next is a bit of modification in the drawer to show the correct component.

~~~javascript
...
<Drawer title="Basic Drawer" onClose={setIsDrawerOpen} open={isDrawerOpen}>
  {drawerPath === "/details" ? <Details1 /> : <Details2 />}
</Drawer>
...
~~~

This can be further improved by a bit of refactoring especially with the magic strings and repeated code and that's it! You can add an _n_ number of drawers as you like.

A small implementation of this is available in [this repository](https://github.com/giltroymeren/exercises-react/commit/134eff8e989daaccbbf2bd398a1bef2d941084dc) for your reference.

## References

- "Drawer - Ant Design". *Ant Design* <[`ant.design/components/drawer`](https://ant.design/components/drawer/)>.
- "Introduction - Ant Design". *Ant Design* <[`ant.design/docs/spec/introduce`](https://ant.design/docs/spec/introduce)>.
- Khatri, Shubham. “How to Compare oldValues and newValues on React Hooks useEffect?” *Stack Overflow*, 3 Nov. 2018, <[`stackoverflow.com/a/53446665`](https://stackoverflow.com/a/53446665)>.
- “useRef – React.” *React*, <[`react.dev/reference/react/useRef`](https://react.dev/reference/react/useRef)>.