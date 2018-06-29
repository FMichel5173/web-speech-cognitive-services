# web-speech-cognitive-services

[![npm version](https://badge.fury.io/js/web-speech-cognitive-services.svg)](https://badge.fury.io/js/web-speech-cognitive-services) [![Build Status](https://travis-ci.org/compulim/web-speech-cognitive-services.svg?branch=master)](https://travis-ci.org/compulim/web-speech-cognitive-services)

Polyfill Web Speech API with Cognitive Services Speech-to-Text service.

This scaffold is provided by [`react-component-template`](https://github.com/compulim/react-component-template/).

# Demo

Try out our demo at https://compulim.github.io/web-speech-cognitive-services?s=your-subscription-key.

We use [`react-dictate-button`](https://github.com/compulim/react-dictate-button/) to quickly setup the playground.

# Background

Web Speech API is not widely adopted on popular browsers and platforms. Polyfilling the API using cloud services is a great way to enable wider adoption. Nonetheless, Web Speech API in Google Chrome is also backed by cloud services.

Microsoft Azure [Cognitive Services Speech-to-Text](https://azure.microsoft.com/en-us/services/cognitive-services/speech-to-text/) service provide speech recognition with great accuracy. But unfortunately, the APIs are not based on Web Speech API.

This package will polyfill Web Speech API by turning Cognitive Services Speech-to-Text API into Web Speech API. We test this package with popular combination of platforms and browsers.

# Test matrix

Browsers are all latest as of 2018-06-28, except:
* macOS was 10.13.1 (2017-10-31), instead of 10.13.5
   * There should be no change on the matrix since Safari does not support Web Speech API
* Xbox was tested on Insider build (1806) with Kinect sensor connected
   * The latest Insider build does not support both WebRTC and Web Speech API, so we suspect the production build also does not support both

Quick grab:

* Web Speech API works on most popular platforms, except iOS
   * iOS: No browsers on iOS support Web Speech API
   * Some platforms requires non-default browser (unsupported in Microsoft Edge)
* Cognitive Services Speech-to-Text work on all popular platforms with their default browsers
   * iOS: Chrome and Edge does not support Cognitive Services because iOS WebView does not support WebRTC

| Platform             | OS                           | Browser              | Cognitive Services (WebRTC) | Web Speech API                          |
| -                    | -                            | -                    | -                           | -                                       |
| PC                   | Windows 10 (1803)            | Chrome 67.0.3396.99  | Yes                         | Yes                                     |
| PC                   | Windows 10 (1803)            | Edge 42.17134.1.0    | Yes                         | No, `SpeechRecognition` not implemented |
| PC                   | Windows 10 (1803)            | Firefox 61.0         | Yes                         | No, `SpeechRecognition` not implemented |
| MacBook Pro          | macOS High Sierra 10.13.1    | Chrome 67.0.3396.99  | Yes                         | Yes                                     |
| MacBook Pro          | macOS High Sierra 10.13.1    | Safari 11.0.1        | Yes                         | No, `SpeechRecognition` not implemented |
| Apple iPhone X       | iOS 11.4                     | Chrome 67.0.3396.87  | No, `AudioSourceError`      | No, `SpeechRecognition` not implemented |
| Apple iPhone X       | iOS 11.4                     | Edge 42.2.2.0        | No, `AudioSourceError`      | No, `SpeechRecognition` not implemented |
| Apple iPhone X       | iOS 11.4                     | Safari               | Yes                         | No, `SpeechRecognition` not implemented |
| Apple iPod (6th gen) | iOS 11.4                     | Chrome 67.0.3396.87  | No, `AudioSourceError`      | No, `SpeechRecognition` not implemented |
| Apple iPod (6th gen) | iOS 11.4                     | Edge 42.2.2.0        | No, `AudioSourceError`      | No, `SpeechRecognition` not implemented |
| Apple iPod (6th gen) | iOS 11.4                     | Safari               | No, `AudioSourceError`      | No, `SpeechRecognition` not implemented |
| Google Pixel 2       | Android 8.1.0                | Chrome 67.0.3396.87  | Yes                         | Yes                                     |
| Google Pixel 2       | Android 8.1.0                | Edge 42.0.0.2057     | Yes                         | Yes                                     |
| Google Pixel 2       | Android 8.1.0                | Firefox 60.1.0       | Yes                         | Yes                                     |
| Microsoft Lumia 950  | Windows 10 (1709)            | Edge 40.15254.489.0  | No, `AudioSourceError`      | No, `SpeechRecognition` not implemented |
| Microsoft Xbox One   | Windows 10 (1806) 17134.4054 | Edge 42.17134.4054.0 | No, `AudioSourceError`      | No, `SpeechRecognition` not implemented |

## Event lifecycle scenarios

We test multiple scenarios to make sure the package polyfill Web Speech API correctly. Following are events and its firing order.

* [Happy path](#happy-path)
* [Abort during recognition](#abort-during-recognition)
* [Network issues](#network-issues)
* [Audio muted or volume too low](#audio-muted-or-volume-too-low)
* [No speech is recognized](#no-speech-is-recognized)
* [Not authorized to use microphone](#not-authorized-to-use-microphone)

### Happy path

Everything works, including multiple interim results.

* Cognitive Services
   1. `RecognitionTriggeredEvent`
   2. `ListeningStartedEvent`
   3. `ConnectingToServiceEvent`
   4. `RecognitionStartedEvent`
   5. `SpeechHypothesisEvent` (could be more than one)
   6. `SpeechEndDetectedEvent`
   7. `SpeechDetailedPhraseEvent`
   8. `RecognitionEndedEvent`
* Web Speech API
   1. `start`
   2. `audiostart`
   3. `soundstart`
   4. `speechstart`
   5. `result` (multiple times)
   6. `speechend`
   7. `soundend`
   8. `audioend`
   9. `result(results = [{ isFinal = true }])`
   10. `end`

### Abort during recognition

#### Abort before first recognition is made

* Cognitive Services
   * Essentially muted the speech, that could still result in success, silent, or no match
* Web Speech API
   1. `start`
   2. `audiostart`
   8. `audioend`
   9. `error(error = 'aborted')`
   10. `end`

#### Abort after some speech is recognized

* Cognitive Services
   * Essentially muted the speech, that could still result in success, silent, or no match
* Web Speech API
   1. `start`
   2. `audiostart`
   3. `soundstart` (optional)
   4. `speechstart` (optional)
   5. `result` (optional)
   6. `speechend` (optional)
   7. `soundend` (optional)
   8. `audioend`
   9. `error(error = 'aborted')`
   10. `end`

### Network issues

Turn on airplane mode.

* Cognitive Services
   1. `RecognitionTriggeredEvent`
   2. `ListeningStartedEvent`
   3. `ConnectingToServiceEvent`
   5. `RecognitionEndedEvent(Result.RecognitionStatus = 'ConnectError')`
* Web Speech API
   1. `start`
   2. `audiostart`
   3. `audioend`
   4. `error(error = 'network')`
   5. `end`

### Audio muted or volume too low

* Cognitive Services
   1. `RecognitionTriggeredEvent`
   2. `ListeningStartedEvent`
   3. `ConnectingToServiceEvent`
   4. `RecognitionStartedEvent`
   5. `SpeechEndDetectedEvent`
   6. `SpeechDetailedPhraseEvent(Result.RecognitionStatus = 'InitialSilenceTimeout')`
   7. `RecognitionEndedEvent`
* Web Speech API
   1. `start`
   2. `audiostart`
   3. `audioend`
   4. `error(error = 'no-speech')`
   5. `end`

### No speech is recognized

Some sounds are heard, but they cannot be recognized as text. There could be some interim results with recognized text, but the confidence is so low it dropped out of final result.

* Cognitive Services
   1. `RecognitionTriggeredEvent`
   2. `ListeningStartedEvent`
   3. `ConnectingToServiceEvent`
   4. `RecognitionStartedEvent`
   5. `SpeechHypothesisEvent` (could be more than one)
   6. `SpeechEndDetectedEvent`
   7. `SpeechDetailedPhraseEvent(Result.RecognitionStatus = 'NoMatch')`
   8. `RecognitionEndedEvent`
* Web Speech API
   1. `start`
   2. `audiostart`
   3. `soundstart`
   4. `speechstart`
   5. `result`
   6. `speechend`
   7. `soundend`
   8. `audioend`
   9. `end`

> Note: the Web Speech API has `onnomatch` event, but unfortunately, Google Chrome did not fire this event.

### Not authorized to use microphone

The user click "deny" on the permission dialog, or there are no microphone detected in the system.

* Cognitive Services
   1. `RecognitionTriggeredEvent`
   2. `RecognitionEndedEvent(Result.RecognitionStatus = 'AudioSourceError')`
* Web Speech API
   1. `error(error = 'not-allowed')`
   2. `end`

# Known issues

* Interim results do not return confidence, final result do have confidence
   * We always return `0.5` for interim results
* Cognitive Services support grammar list but not in JSGF format, more work to be done in this area
   * Although Google Chrome support setting the grammar list, it seems the grammar list is not used at all

# Contributions

Like us? [Star](https://github.com/compulim/web-speech-cognitive-services/stargazers) us.

Want to make it better? [File](https://github.com/compulim/web-speech-cognitive-services/issues) us an issue.

Don't like something you see? [Submit](https://github.com/compulim/web-speech-cognitive-services/pulls) a pull request.
