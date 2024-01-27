---
layout: post
title: "How to show an antd drawer within a drawer"
date: 2024-01-27
categories: notes javascript typescript react antd
---

This is related to a [previous post]({{ site.baseurl }}{% post_url 2024-01-13-how-to-attach-an-antd-drawer-to-a-direct-url-in-react %}) that dealt with antd drawers and react-router. I had another requirement to call an drawer inside a drawer wherein both need to shown in the UI.

## Table of Contents
- [Table of Contents](#table-of-contents)
- [Challenge](#challenge)
- [Solution](#solution)
- [What's next?](#whats-next)
- [References](#references)

## Challenge

The product I was working on had a design direction to use drawers for forms in order to avoid redirecting users to another page. This means certain actions such as addition and update of sub-elements in these forms, that used to appear in that same separate page, had to be within the bounds of the drawer now.

This was an exciting challenge and we experimented with modals, accordions and had a conclusion that showing another drawer, while inside a drawer, is the best solution in our use case.

## Solution

This is a straightforward solution that works in the same principle as before: add a `Drawer` component and attach a toggle function to it. 

I also learned that this can be, in theory, be replicated many times with child drawers calling their own child drawers and so on. It just needs each drawer content to be declared as its own component.

~~~javascript
const ParentComponent: React.FC = () => {
  const [isDrawerOpen, setIsDrawerOpen] = React.useState(false);
  const toggleDrawer = () => setIsDrawerOpen(!isDrawerOpen);

  return (
    <>
      <strong>Title</strong>
      <p>...</p>
    
      <Button type="primary" onClick={toggleDrawer}>
        Learn more
      </Button>

      <Drawer
        placement="right"
        onClose={toggleDrawer}
        open={isDrawerOpen}
        width="50%"
      >
        {<ChildComponent />}
      </Drawer>
    </>
  )
}
~~~

In this example, `ChildComponent` can have its own drawer by using the same structure as `ParentComponent` and this can be done multiple times.

 I have managed to go down two child levels and decreased the width of each parent by 10% to maintain the effect that they are on top of each other. Otherwise they will all just cover the entire width of their respective parent. 

![Drawers within drawers](/images/posts/2024-01-27-how-to-show-an-antd-drawer-within-a-drawer/drawers-in-drawers.png){:class="img-responsive"}

## What's next?

This is a neat trick for niche design and interaction requirements. However it can become tedious to declare a new `Drawer` in each component to hold whatever we want. 

It would be great to put it in a useful React hook that will just expose needed methods to toggle its visibility while maintaining its width and uniqueness among the other drawers and the rest of DOM elements.

## References

- "Drawer - Ant Design". *Ant Design* <[`ant.design/components/drawer`](https://ant.design/components/drawer/)>.
- Ibrahim, Salar. “Ability to open Drawer inside Modal, div or any container not just htmlBody · Issue #11671 · Ant-Design/Ant-Design.” *GitHub*, <[`github.com/ant-design/ant-design/issues/11671`](https://github.com/ant-design/ant-design/issues/11671)>.
