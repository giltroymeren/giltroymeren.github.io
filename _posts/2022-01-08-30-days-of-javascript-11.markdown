---
layout: post
title:  "Notes on Day 11 of 30 Days of JavaScript"
date:   2022-01-08
categories: notes series javascript javascript30
---

The lesson mostly emphasized how to manipulate the browser's native video player's design and behavior.

## Table of Contents
* TOC
{:toc}

## Lessons Learned

### `HTMLMediaElement.play()`

*This method attempts to begin playback of the media. It returns a Promise which is resolved when playback has been successfully started*.

*Failure to begin playback for any reason, such as permission issues, result in the promise being rejected*.

~~~ javascript
const promise = HTMLMediaElement.play();

const video = document.getElementById('banner-video');
video.addEventListener('play', handlePlay);
~~~

### `HTMLMediaElement.pause()`

*The HTMLMediaElement.pause() method will pause playback of the media, if the media is already in a paused state this method will have no effect.*

~~~ javascript
HTMLMediaElement.pause();

const video = document.getElementById('banner-video');
video.addEventListener('pause', handlePause);
~~~

### `HTMLMediaElement: timeupdate` event

*The timeupdate event is fired when the time indicated by the currentTime attribute has been updated*.

~~~ javascript
const video = document.getElementById('banner-video');
video.addEventListener('timeupdate', handleProgress);

video.ontimeupdate = () => handleProgress();
~~~

## References
* "Custom HTML5 Video Player" *wesbos.com*, uploaded by Wes Bos, 8 Dec. 2016, <[`javascript30.com`](https://javascript30.com/)>.
* "HTMLMediaElement.pause() - Web APIs \| MDN." *MDN Web Docs*, Mozilla Web Network, 15 Sep. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/pause`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/pause)>.
* "HTMLMediaElement.play() - Web APIs \| MDN." *MDN Web Docs*, Mozilla Web Network, 15 Oct. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/play`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/play)>.
* "HTMLMediaElement: timeupdate event - Web APIs \| MDN." *MDN Web Docs*, Mozilla Web Network, 21 Sep. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/timeupdate_event`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/timeupdate_event)>.
