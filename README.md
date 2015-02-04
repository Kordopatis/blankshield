# blankshield

Prevent [reverse tabnabbing](https://sites.google.com/site/bughunteruniversity/nonvuln/phishing-with-window-opener)
based phishing attacks that take advantage of _blank targets.

* [Overview](#overview)
* [Solutions](#solutions)
* [Installation](#installation)
* [Caveats](#caveats)

## Overview

Tabs or windows opened using JavaScript or `target="blank"` have some limited
access to the parent window, ignoring cross-origin restrictions. Among that
is the ability to redirect the parent tab or window using
`window.opener.location`.

While it may seem harmless, a phishing attack is possible when web applications
permit or make use of user-submitted anchors with `target="blank"` or
`window.open()`. Consider the following scenario:

You're an admin using some forum or chat web software. You're currently logged
into the app, and view a message left by a user. The user asks or convinces
you to click a link in his message, which opens in a new tab. While the new
page may look completely safe - perhaps just a screenshot or bug report in some
HTML, it executes the following JS:

``` JavaScript
window.opener.location.assign('https://yourcompanyname.phishing.com');
```

What you don't realize is that while dealing with this illegitimate customer or
user complaint, your application's tab was redirected in the background. To
what? An identical phishing website, simply requesting that you enter your
credentials to log back in.

Is there a chance you might not check the URL? That you didn't notice the tab
icon refresh? While many are suspicious of links they click and new tabs they
open - what about existing tabs?

## Solutions

A handful of solutions exist to prevent this sort of attack. You could:

* Remove or disallow `target="blank"` for any anchors pointing to a
different origin.
* Append `rel="noreferrer"` to any links with `target="blank"`. When done,
  `window.opener` will be null from the child window. It's well supported among
  webkit-based browsers, though you'll fall short with IE and even newer
  releases of FireFox. And of course, it prevents sending the referrer in
  the request headers. You could fall off as an identifiable source of traffic
  for some friendly sites.
* Listen for the click event and prevent the default browser behavior of
  opening a new tab. Then, open a new window with the href and set its opener
  to null. This is what blankshield does.

## Installation

The library can be installed via npm:

``` bash
npm install blankshield
```

Or using bower:

``` bash
bower install blankshield
```

## Usage

blankshield.js, which works in global, CommonJS and AMD contexts, exports
a single function: `blankshield()`.

``` JavaScript
// It works on a single element
blankshield(document.getElementById('some-anchor'));

// Array-like objects such as HTMLCollections
blankshield(document.getElementsByClassName('user-submitted-link'));
blankshield(document.getElementsByTagName('a'));

// As well as jQuery
blankshield($('a[target=_blank]'));
```

Its effectiveness will depend on your integration.

``` JavaScript
// Don't bind listeners to the anchors that would stop event propagation.
// In the example below, blankshield is not able to intercept the click
// behavior.
var anchor = document.getElementById('some-anchor')
anchor.addEventListener('click', function(e) {
   return e.stopImmediatePropagation();
});
blankshield(document.getElementById('some-anchor'));
```

## Caveats

This library only helps make it easier to prevent reverse tabnabbing which takes
advantage of `target="blank"`. However, it can't help with other elements
or behavior that calls, for example, `window.open()`.
