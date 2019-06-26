# Back Button Media Query Explainer

Author: Jay Harris <harrisjay@chromium.org>

Date: 2019-01-23

Note: This explainer is based on [mgiuca](https://github.com/mgiuca)'s proposal on [w3c/manifest#693](https://github.com/w3c/manifest/issues/693).

## Background

Web applications, and more specifically, Progressive Web Applications, can be "installed" (or "added to the home screen"),
implying that with the `display-mode`s of `"standalone"`, `"fullscreen"`, and `"minimal-ui"`
they then occupy the whole screen on mobile form factor devices,
or run in a windowed (yet full-screenable) context on desktop form factor devices.
Different user agents have different navigation paradigms, especially when it comes to back/forward navigation,
and potentially support additional navigation gestures or possess operating system level back buttons
(historically on Android implemented as hardware buttons), but typically do not have forward buttons.
This causes issues in the "installed" scenario described above, leading to potentially two navigation affordances:
those provided by the user agent, and those provided by the application, or even worse: no navigation affordances.

## Overview

The **back button media query** is a proposed API (as defined by the [Web App Manifest Standard](https://www.w3.org/TR/appmanifest/)) to determine whether a back button is currently being shown.

This API is being proposed as a way to eliminate the "double back button" problem (seen [here](images/double-back-button.jpg), on Twitters PWA in the Microsoft store). This problem arises because developers of [`"standalone"`, `"fullscreen"`, and `"minimal-ui"`](https://www.w3.org/TR/appmanifest/#display-modes) apps have no way of determining whether a back button will be provided by the user agent (the browser, OS, or hardware), and so, are forced to implement their own, to ensure that their app is functional without the browser's UI.

This will allow apps to avoid the "double back button" problem by letting them conditionally display a back button depending on whether the user agent is already displaying one. Adding a standard way to detect this means that we can avoid user-agent and viewport-size workarounds, which are not ideal for web compatibility.

Possible Areas of Expansion
- Detection of other controls, such as a forward button, browser tab strip, or share button.

Examples of sites that may use the API
- Twitter (currently they are using user-agent detection to determine whether to show a back button).
- Any app that performs navigations.

Advantages over user-agent detection
- If the user agent changes whether a `display-mode` has a back button the site will not have to be updated.
- Sites don't have to understand how different user-agents will render them on different devices in different display-modes. For example:
    - PWAs on Edge display a back button if there are pages in history.
    - PWAs on Chrome Desktop never show a back button.
    - PWAs on Android always have a back button available.

## API Proposal

A CSS Media query `navigation-controls` which determines what navigation controls are currently available, with the possible values `'none'` | `'back'`, and the possibility of being extended with `'back-and-forward'` in future. However, the API is designed to be used in a [boolean context](https://www.w3.org/TR/mediaqueries-4/#mq-boolean-context), so all future values should imply the presence of a back button (e.g. we wouldn't add a separate `'forward'` value).

*Note: This value is an enum not a boolean because `<mq-boolean>` is intended for [legacy purposes only](https://www.w3.org/TR/mediaqueries-4/#grid), with the spec noting that "If this feature were being designed today, it would instead use proper named keywords for its values."*

There is precedence for doing feature detection via a CSS media query in the Web App Manifest's [display mode](https://www.w3.org/TR/appmanifest/#the-display-mode-media-feature), which allows developers to determine how their app is being displayed.

The `navigation-controls` query should always be available, including when the app is not in a standalone window (i.e. it is in a normal browser window). In such a scenario, it should have an appropriate value, likely `'back'`. This has the added advantage that `navigation-controls` can be consulted independently of `display-mode`.

### CSS

It is expected that for the majority of use cases, CSS detection should be sufficient. This has the added advantage that it will work even if JavaScript is not enabled on the page (though it is unlikely that much else in a modern web application will).

```css
@media (navigation-controls) {
    #back-button {
        display-none;
    }
}
```

### JavaScript
In more complicated scenarios, it might be useful to detect the presence of the back button in JavaScript. Fortunately there is already a mechanism for running media queries in JavaScript.

```js
const backButtonQuery = window.matchMedia('navigation-controls');
const backButton = window.getElementById('back-button');
backButtonQuery.addEventListener('change', query => {    
    if (query.matches) {
        myFancyAnimateBackButtonIn(backButton);
    } else {
        myFancyAnimateBackButtonOut(backButton);
    }
});
```

*Note: It is not enough to simply check for the presence of a back button once, you must add the event listener to the media query, as it is  possible for the back button to be dynamically added and removed by the user agent. For example, in Chrome Desktop, an app can be moved into a tab and out again into an app window, and in Edge, the back button is hidden when there is no history.*

## Specific Operating System Treatment

In general, operating systems go one of two ways:

1. The system is expected to provide a back button, but may suppress it if unnecessary (e.g. Window 10, where a back button is added to PWAs if there is history).
2. There is no 'system' back button, so applications are always responsible for adding their own back button (e.g. iOS).

### Not Android
On non-Android platforms, there is no hardware/OS level button, so the decision to show or hide a back button can be made on a per user-agent per platform level and reported via the query. Below are some recommendations.

- Windows 10: Display a user-agent back button when history is available. This is consistent with existing applications from the Microsoft store. *Note: There may be a quirk here if the value of `navigation-controls` changes on Windows 10 when there is no history.* ![Windows 10 Twitter PWA](images/win-10-twitter-navigation-buttons.jpg)
- macOS: Unclear what the OS conventions are. iTunes displays navigation buttons outside of the title bar ![iTunes navigation buttons](images/osx-itunes-navigation-buttons.png) while the App Store displays them inside the title bar. ![AppStore navigation buttons](images/osx-appstore-navigation-buttons.png) Both apps display navigation buttons even when there is no history (unlike Windows 10).
- iOS: Don't display a user-agent back button. iOS applications are generally responsible for drawing their own navigation buttons. ![iOS iMessage navigation button](images/ios-imessage-navigation-buttons.jpg). On newer iPhones there are swipe gestures as an OS convention for going back. It is unclear whether the availability of these gestures should count as the presence of a back button.
- Linux: Unclear what conventions are.
- Chrome OS: Unclear what conventions are.

### Android
Android is unique in that it provides an OS level back button but also recommends that applications show their own back button. ![Android Play Store](images/android-playstore-navigation-buttons.jpg)

It is believed that Twitter is leaning into this, to make their application seem more native.

Normally, we would hesitate to recommend user-agent detection, however, in this situation it does seem appropriate, as the intended behavior is "Display a back button if the system doesn't provide one *OR* if the operating system is Android." As developers are explicitly thinking about Android, it's not unreasonable to explicitly mention it in code.

An alternative would be to suggest user-agents always return `navigation-controls: none` on Android, under the assumption that apps will always wish to show controls on Android. However, this might be overly broad.

## Questions and Concerns

### Why isn't this a JavaScript API?
- Doesn't require JavaScript to be enabled to work.
- We don't want to prescribe how developers do things; as a media query you can access this API from CSS or JavaScript.
- An API of the form
    ```js
    if (navigator.features.backButton) {
        // stuff
    }
    ```
   isn't sufficient, as we need some way of subscribing to change events, as the user agent may show/hide the back button (for example, changing between tablet and desktop mode in Window 10/Chrome OS or undocking/docking an app window). We get this for free with media queries. A notable exception is [`navigator.onLine`](https://developer.mozilla.org/en-US/docs/Web/API/NavigatorOnLine/onLine) that supports event listeners.

### Why can't I just say in my manifest whether I want a back button?

We're considering this, but specs generally prefer not to tell browsers how their UI should look. This way, you can respond to what the browser has done in an appropriate way.

### Can I tell what kind of button is being displayed (i.e. hardware or software; from the user-agent or from the OS)?

This question came up on a github [issue](https://github.com/w3c/manifest/issues/693#issuecomment-402608547). Because Android has different semantics associated with a back button press to other operating systems (e.g. close a modal when the back button is pressed, instead of going back a page), it would be useful to know the type of back button that is being provided. However, this issue seems to be more about "Is this Android" than "Is there an OS back button", so again, user agent detection seems like a more appropriate solution.
