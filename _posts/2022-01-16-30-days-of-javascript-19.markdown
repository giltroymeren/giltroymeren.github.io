---
layout: post
title:  "Notes on Day 19 of 30 Days of JavaScript"
date:   2022-01-16
categories: notes series javascript javascript30 video
---

## Table of Contents
* TOC
{:toc}

## Lessons Learned

### Navigator

*The `window.navigator` property refers to a Navigator object that contains information about the web browser as a whole, such as the version and a list of the data formats it can display. The Navigator object is named after Netscape Navigator, but it is also supported by Internet Explorer.*

The following are some of its properties that describe certain aspects of the browser in use:

- `appName`* - the official name of the browser.
- `appVersion`* - the officil version of the browser.
- `userArgent` - the user agent string for the current browser. 

    *The string that the browser sends in its `USER-AGENT` HTTP header. This property typically contains all the information in both appName and appVersion.*

- `appCodeName`* - the internal "code" name of the current browser.
- `platform`* - the hardware platform of the browser.

    \* Deprecated properties, do not use.

### `Navigator.mediaDevices`

A *read-only property returns a `MediaDevices` object, which provides access to connected media input devices like cameras and microphones, as well as screen sharing*.

~~~ javascript
const mediaDevices = navigator.mediaDevices;
~~~

### `Navigator.mediaDevices.getUserMedia()`

This method returns a `Promise` that resolves to any of the following `MediaStream` objects: *a video track (produced by either a hardware or virtual video source such as a camera, video recording device, screen sharing service, and so forth), an audio track (similarly, produced by a physical or virtual audio source like a microphone, A/D converter, or the like), and possibly other track types*.

~~~ javascript
const video = document.getElementById("video-container");

const async getVideoStream = () => {
  navigator.mediaDevices.getUserMedia({ video: true, audio: false })
    .then(localMediaStream => {
      video.srcObject = localMediaStream;
      video.play();
    }).catch(error => {
      console.error(`Error occurred: `, error);
    });
}
~~~

### `URL.createObjectURL()`

This is a *static method creates a `DOMString` containing a URL representing the object given in the parameter*.

~~~ javascript
const objectURL = window.URL.createObjectURL(object)
~~~

### `HTMLMediaElement: canplay` event

This event executes when the user agent can play the media.

~~~ javascript
const video = document.getElementById("video-container");

video.addEventListener('canplay', playVideo)
~~~

## References
* "URL.createObjectURL() - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 15 Sep. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/URL/createObjectURL`](https://developer.mozilla.org/en-US/docs/Web/API/URL/createObjectURL)>.
* "JavaScript: The Definitive Guide, Fourth Edition." *O’Reilly Online Learning*, <[`oreilly.com/library/view/javascript-the-definitive/0596000480/ch13s06.html`](https://www.oreilly.com/library/view/javascript-the-definitive/0596000480/ch13s06.html)>. Accessed 15 Jan. 2022.
* "MediaDevices.getUserMedia() - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 30 Nov. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/MediaDevices/getUserMedia`](https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices/getUserMedia)>.
* "Navigator - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 15 Sep. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/Navigator`](https://developer.mozilla.org/en-US/docs/Web/API/Navigator)>.
* "Unreal Webcam Fun" *wesbos.com*, uploaded by Wes Bos, 8 Dec. 2016, <[`javascript30.com`](https://javascript30.com/)>.
