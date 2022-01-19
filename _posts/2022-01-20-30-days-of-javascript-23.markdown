---
layout: post
title:  "Notes on Day 23 of 30 Days of JavaScript"
date:   2022-01-20
categories: notes series javascript javascript30
---

## Table of Contents
* TOC
{:toc}

## Lessons Learned

### `SpeechSynthesisUtterance`

*The `SpeechSynthesisUtterance` interface of the Web Speech API represents a speech request. It contains the content the speech service should read and information about how to read it (e.g. language, pitch and volume.)*

~~~ javascript
const synthesizer = new SpeechSynthesisUtterance();
~~~

It has the following properties all related to speech and audio:

- `SpeechSynthesisUtterance.lang` - the language of the utternace.
- `SpeechSynthesisUtterance.pitch` - the pitch at which the utterance will be spoken at.
- `SpeechSynthesisUtterance.rate` - the speed at which the utterance will be spoken at.
- `SpeechSynthesisUtterance.text`
- `SpeechSynthesisUtterance.voice` - the voice that will be used to speak the utterance (specifically the voiceover, tied to a nationality).
- `SpeechSynthesisUtterance.volume`

### `SpeechSynthesis.getVoices()`

*The `getVoices()` method of the `SpeechSynthesis` interface returns a list of `SpeechSynthesisVoice` objects representing all the available voices on the current device*.

This is usually used to populate a dropdown of available voice options to choose from.

~~~ javascript
speechSynthesis
>  SpeechSynthesis
    onvoiceschanged: null
    paused: false
    pending: false
    speaking: false
    [[Prototype]]: SpeechSynthesis

speechSynthesis.getVoices()
> 0: SpeechSynthesisVoice {voiceURI: 'Alex', name: 'Alex', lang: 'en-US', localService: true, default: true}
  1: SpeechSynthesisVoice {voiceURI: 'Alice', name: 'Alice', lang: 'it-IT', localService: true, default: false}
  2: SpeechSynthesisVoice {voiceURI: 'Alva', name: 'Alva', lang: 'sv-SE', localService: true, default: false}
  3: SpeechSynthesisVoice {voiceURI: 'Amelie', name: 'Amelie', lang: 'fr-CA', localService: true, default: false}
  ...
~~~

### `SpeechSynthesis: voiceschanged` event

In relation to the `getVoices()` method, this event fires when a change in the `SpeechSynthesisVoice` happened.

~~~ javascript
function setVoices() {
  const voices = speechSynthesis.getVoices();
  const selectField = document.getElementById("voices-dropdown");
  voices
    .map(voice => {
      const option = document.createElement('option');
      option.textContent = `${voice.name} (${voice.lang})`;
      selectField.appendChild(option)
    });
}

speechSynthesis.addEventListener('voiceschanged', setVoices);
~~~

## References
* "SpeechSynthesis.getVoices() - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 17 Dec. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/SpeechSynthesis/getVoices`](https://developer.mozilla.org/en-US/docs/Web/API/SpeechSynthesis/getVoices)>.
* "SpeechSynthesisUtterance - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 23 Nov. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/SpeechSynthesisUtterance`](https://developer.mozilla.org/en-US/docs/Web/API/SpeechSynthesisUtterance)>.
* "SpeechSynthesis: voiceschanged event - Web APIs \| MDN." *MDN Web Docs*, Mozilla Developer Network, 17 Dec. 2021, <[`developer.mozilla.org/en-US/docs/Web/API/SpeechSynthesis/voiceschanged_event`](https://developer.mozilla.org/en-US/docs/Web/API/SpeechSynthesis/voiceschanged_event)>.
* "Speech Synthesis" *wesbos.com*, uploaded by Wes Bos, 8 Dec. 2016, <[`javascript30.com`](https://javascript30.com/)>.
