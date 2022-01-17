---
layout: post
title:  "Notes on Day 21 of 30 Days of JavaScript"
date:   2022-01-18
categories: notes series javascript javascript30 geolocation
---

## Table of Contents
* TOC
{:toc}

## Lessons Learned

### `Navigator.geolocation`

*The `Navigator.geolocation` read-only property returns a `Geolocation `object that gives Web content access to the location of the device. This allows a Web site or app to offer customized results based on the user's location.*

~~~ javascript
const geolocator = navigator.geolocation;
~~~

### `Geolocation.watchPosition()`

*The `Geolocation` method `watchPosition()` method is used to register a handler function that will be called automatically each time the position of the device changes. You can also, optionally, specify an error handling callback function*.

~~~ javascript
const geolocator = navigator.geolocation;
geolocator.watchPosition((data) => {
  console.log(data.coords.latitude);
  console.log(data.coords.longitude);
}, (error) => {
  console.error('Error occurred: ', error);
});
~~~

The data returned is of a `GeolocationPosition` type with the following structure:

~~~ javascript
GeolocationPosition
  coords: GeolocationCoordinates
    accuracy: 50.0
    altitude: null
    altitudeAccuracy: null
    heading: null
    latitude: 11.11
    longitude: 11.11
    speed: null
    [[Prototype]]: GeolocationCoordinates
  timestamp: 11111111111111
  [[Prototype]]: GeolocationPosition
~~~

## References
* "Geolocation.watchPosition() - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 11 Oct. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/Geolocation/watchPosition`](https://developer.mozilla.org/en-US/docs/Web/API/Geolocation/watchPosition)>.
* "Navigator.geolocation - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 15 Sep. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/Navigator/geolocation`](https://developer.mozilla.org/en-US/docs/Web/API/Navigator/geolocation)>.
* "Geolocation based Speedometer and Compass" *wesbos.com*, uploaded by Wes Bos, 8 Dec. 2016, <[`javascript30.com`](https://javascript30.com/)>.
