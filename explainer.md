# Back Button Media Query Explainer

Author: Jay Harris <harrisjay@chromium.org>

Date: 2019-01-23

## Overview

The **back button media query** is a proposed API (as defined by the [Web App Manifest Standard](https://www.w3.org/TR/appmanifest/)) to determine whether a back button is currently being shown.

This API is being proposed as a way to eliminate the "double back button" problem (seen [here](images/double-back-button.jpg), on Twitters PWA in the Microsoft store). This problem arises because developers of "standalone" [LINK](), and "minimal-ui" [LINK]() apps have no way of determining whether a back button will be provided by the user agent (in the browser, OS or hardware), and so, are forced to implement their own, to ensure users are still able to use the app.

This will allow apps to avoid the "double back button" problem by letting them conditionally display a back button depending on whether the user agent is already displaying one. Adding a standard way to detect this means that we can avoid user-agent and viewport-size workarounds, which are not ideal for Web Compatibility.

Possible Areas of Expansion
- Detection of other controls, such as a forward button or tab strip

Examples of sites that may use the API
- Twitter (currently they are using user-agent detection to determine whether to show a back button).
- Any app that performs navigations.

Advantages over user-agent detection
- If the user agent changes whether a display-mode displays a back button the site will not have to be updated.
- Sites don't have to understand how different user-agents will render them on different devices in different display-modes. For example, PWAs on Edge display a back button if there are pages in history while PWAs in Chrome never show a back button and PWAs on Android always have a back button available.

## API Proposal

## Specific Operating System Treatment

## Other Solutions

### Javascript API

### Requesting a Back Button

## Questions and Concerns