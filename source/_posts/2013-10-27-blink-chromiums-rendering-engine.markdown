---
layout: post
title: "Blink: Chromium's Rendering Engine"
date: 2013-10-27 07:50
comments: true
categories:  chromium
---

[Blink](http://www.chromium.org/blink) is Google's fork of the
[WebKit browser engine](https://www.webkit.org/), which in turn is Apple's fork
of KDE's KHTML engine.


# WebKit Legacy

WebKit (the precursor of Blink) was designed to be platform-independent, and
was successfully ported to many platforms, such as Mac OS and iOS
(using Cocoa), Windows, Linux (using GTK), and Android.
Chromium was also designed to be ported to multiple platforms, and adds another
platform-independent layer on top of the Blink rendering engine, called the
content module. So, from the WebKit code perspective, Blink sits on top of a
single platform, called "chromium".

The WebKit project contains a rendering engine (WebCore), a JavaScript engine
(JavaScriptCore), and an embedding layer. Blink adopted WebCore, but uses
the [V8 JavaScript Engine](https://code.google.com/p/v8/) instead of
JavaScriptCore, and dropped the embedding layer in favor of having its content
layer talk directly to WebCore.


# Code Layout

Blink's source code is at
[`third_party/WebKit`](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/)
in the [Chromium source tree](http://cs.chromium.org). The top-level
directories are as follows.

* [`public/`](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/public/)
  contains the header files makeing up the API between Blink and Chromium's
  content layer.
* [`Source`](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/Source/)
  contains the actual Blink source code.
* [`LayoutTests`](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/LayoutTests/)
  has automated tests for the engine's features.
* [`Tools`](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/Tools/)
  contains scripts for building and running the code.
  [`Tools/Scripts`](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/Tools/)
  is heavily referenced in WebKit's old

I have not explored the
[`PerformanceTests`](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/PerformanceTests/)
or
[`ManualTests`](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/ManualTests/)
directories. They appear to be described by the
[WebKit Wiki's Performance Tests page](http://trac.webkit.org/wiki/Performance%20Tests)
and
[WebKit Wiki's Manual Testing page](http://trac.webkit.org/wiki/Performance%20Tests).

At the time of this writing,
[WebKit's source tree](https://trac.webkit.org/browser/trunk/) has the same
top-level structure. Comparing the `Source/` directories shows the top-level
differences between Blink and WebKit.

## The Public API

The headers in
[`public/`](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/public/)
are relatively well-commented, so they make a great starting point for
understanding a part of Chromium.

The public APIs fall into two big categories.

* [`public/web`](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/public/web/)
contains the API implemented by Blink and called by Chromium's content layer
* [`public/platform`](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/public/platform/)
has the lower-level API used by Blink and implemented by the content layer

The classes in the public APIs usually have easy-to-guess corresponding classes
in the implementation. For example, the `WebView` abstract class, defined in
[`public/web/WebView.h`](http://www.chromium.org/developers/testing/webkit-layout-tests),
has one concrete implementation, `WebViewImpl`, which is defined in
[`Source/web/WebViewImpl.h`](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/Source/web/WebViewImpl.h).
[`Source/web/WebViewImpl.cpp`](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/Source/web/WebViewImpl.cpp)
contains both the `WebViewImpl` implementation, as well as the definitions for
`WebView`'s static members.

## The Source Code

Blink's implementation is split into the following top-level directories.

* [`Source/web`](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/Source/web/)
  has the implementations of the public APIs in
  [`public/web`](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/public/web/)
* [`Source/core`](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/Source/core/)
  has the WebCore implementation, and is the meat of the project
* [`Source/platform`](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/Source/platform/)
  contains some lower-level code used by WebCore
* [`Source/modules`](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/Source/modules/)
  has implementations for some W3C specifications; for example, the
  [Geolocation API Spefication](http://dev.w3.org/geo/api/spec-source.html) is
  implemented in
  [`Source/modules/geolocation`](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/Source/modules/geolocation/)
* [`Source/bindings`](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/Source/bindings/)
  has the code that produces the glue connecting JavaScript code running inside
  the browser and WebCore; for example, this is what makes
  [document.querySelectorAll](https://developer.mozilla.org/en-US/docs/Web/API/Document.querySelectorAll)
  inside JavaScript call into the
  [WebCore::Node::querySelectorAll](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/Source/core/dom/Node.cpp&q=querySelectorAll)
  method in WebCore
* [`Source/inspector`](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/Source/web/)
  contains the specification of the
  [Chrome remote debugging protocol](https://developers.google.com/chrome-developer-tools/docs/debugger-protocol),
  and the source code for the
  [Chrome DevTools](https://developers.google.com/chrome-developer-tools/),
  which are implemented as a Web application that uses the remote debugging
  protocol
* [`Source/testing`](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/Source/testing/)
  has the source for the special binary used to run the Blink layout tests;
  this binary has additional JavaScript interfaces exposing Blink internals, to
  facilitate testing
* [`Source/build`](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/Source/build/)
  has an assortment of scripts used by the build process


## The Layout Tests

Blink's regression tests are Web pages that are loaded in a content layer that
is hacked to output textual representations of the pages. This is really
convenient, because a Web application developer can contribute a failing test
for a bug or feature without having to learn C++ or Blink's inner workings.

To get a taste of what tests look like, skim
[LayoutTests/fast/xmlhttprequest-get.xhtml](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/LayoutTests/fast/xmlhttprequest/xmlhttprequest-get.xhtml)
and the corresponding
[LayoutTests/fast/xmlhttprequest-get-expected.txt](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/LayoutTests/fast/xmlhttprequest/xmlhttprequest-get-expected.txt).


# Building and Testing

The `all_webkit` target contains the Blink-related deliverables.

```bash
ninja -C out/Debug all_webkit
```

Blink cannot run on its own, due to its dependency on Chromium's content layer.
The *Content Shell* offers a rudimentary UI on top of the content layer, and is
a low-overhead way of getting Blink up and running.

```bash
# Linux
out/Debug/content_shell

# OS X
out/Debug/Content\ Shell.app/Contents/MacOS/Content\ Shell
```


# Further Reading

At the time of this writing, the
[Technical Articles in the WebKit blog](https://www.webkit.org/coding/technical-articles.html)
and many of the entries on the
[WebKit Wiki](http://trac.webkit.org/wiki) are still relevant and worth the
time spent reading them.


# References

* [Blink Public C++ API](http://www.chromium.org/blink/public-c-api)
* [The Chromium Content Module](http://www.chromium.org/developers/content-module)
* [Getting around the Chrome Source Code](http://www.chromium.org/developers/how-tos/getting-around-the-chrome-source-code)
