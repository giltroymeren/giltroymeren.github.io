---
layout: post
title:  "Notes on Day 5 of 30 Days of JavaScript"
date:   2022-01-02
categories: notes series javascript css 30_days_of_js
---

## Table of Contents
* TOC
{:toc}

## Lessons Learned

### Flex alignment properties

These four alignment properties of CSS flex might be confusing at first but once we get familiar with their characteristics, it will be easier to use them in your projects.

| Property          | Definition                                                                        | (Basic) Values                                                                 |
|-------------------|-----------------------------------------------------------------------------------|--------------------------------------------------------------------------------|
| `align-content`   | Aligns items on the parallel axis (x-axis) of the flex container.                 | `flex-start`, `flex-end`, `center`, `space-between`, `space-around`, `stretch` |
| `justify-content` | Justifies items on the parallel axis (x-axis) of the flex container.              | `flex-start`, `flex-end`, `center`, `space-between`, `space-around`, `stretch` |
| `align-items`     | Aligns items on the perpendicular axis (y-axis) of the flex container.            | `auto`, `flex-start`, `flex-end`, `center`, `left`, `right`                    |
| `justify-items`   | Justifies items on the perpendicular axis (y-axis) of the flex container.         | `auto`, `flex-start`, `flex-end`, `center`, `left`, `right`                    |
| `align-self`      | Similar to `align-items` but overrides it and applies on an individual level.     | `auto`, `flex-start`, `flex-end`, `center`, `baseline`, `stretch`              |
| `justify-self`    | Similar to `justify-items` but overrides it and justifies on an individual level. | `auto`, `flex-start`, `flex-end`, `center`, `baseline`, `stretch`              |

### `DOMTokenList.toggle()` 

This method adds a token if it doesn't exist in the list; otherwise it will remove the token.
It is usually used in manipulating CSS classes of elements.

~~~ css
nav li {
  background: #F9F9F9;
  color: #292B30
}
nav li.active {
  background: #292B30;
  color: #F9F9F9;
}
~~~

~~~ javascript
document.querySelectorAll('nav li')
  .forEach(item => item.addEventListener('click', this.classList.toggle('active')))
~~~

## References
* "align-self
 - CSS: Cascading Style Sheets \| MDN." *MDN Web Docs*, Mozilla Developer Network, 29 Oct. 2021, <[`developer.mozilla.org/en-US/docs/Web/CSS/align-self`](https://developer.mozilla.org/en-US/docs/Web/CSS/align-self)>.
* "Bukhvalova, Yulia. “Flex Cheatsheet - Alignment." *Про CSS (About CSS)*, <[`yoksel.github.io/flex-cheatsheet/#section-alignment`](https://yoksel.github.io/flex-cheatsheet/#section-alignment)>. Accessed 30 Dec. 2021.
* "CSS Flexible Box Layout Module Level 1 - Alignment." *W3C*, W3C, 19 Nov. 2018,<[`w3.org/TR/css-flexbox-1/#alignment`](https://www.w3.org/TR/css-flexbox-1/#alignment>).
* "DOMTokenList.toggle() - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 22 Nov. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/DOMTokenList/toggle`](https://developer.mozilla.org/en-US/docs/Web/API/DOMTokenList/toggle)>.
* "Flex Panels Image Gallery." *wesbos.com*, uploaded by Wes Bos, 8 Feb. 2016, <[`javascript30.com`](https://javascript30.com/)>.
* "justify-items - CSS: Cascading Style Sheets \| MDN." *MDN Web Docs*, Mozilla Developer Network, 29 Oct. 2021, developer.<[`mozilla.org/en-US/docs/Web/CSS/justify-items`](https://developer.mozilla.org/en-US/docs/Web/CSS/justify-items)>.
* "justify-self - CSS: Cascading Style Sheets \| MDN." *MDN Web Docs*, Mozilla Developer Network, 29 Oct. 2021, developer. <[`mozilla.org/en-US/docs/Web/CSS/justify-self`](https://developer.mozilla.org/en-US/docs/Web/CSS/justify-self)>.