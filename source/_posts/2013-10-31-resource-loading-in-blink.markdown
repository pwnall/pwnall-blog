---
layout: post
title: "Resource Loading in Blink"
date: 2013-10-31 14:25
comments: true
published: false
categories: chromium
---

This article documents my exploration of Blink's logic for loading resources.
Blink is in
[`third_party/WebKit`](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/)
in the Chromium codebase, and all the paths in
this article are relative to that directory.

The interesting code is in the
[`Source/core/fetch/`](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/Source/core/fetch/)
and
[`Source/core/loader/`](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/Source/core/loader/)
directories.


# Loading

The loader interface is in
[`Source/core/loader/ThreadableLoader.h`](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/Source/core/loader/ThreadableLoader.h),
and is a concise introduction to the loader's world. The associated
implementation file,
[`Source/core/loader/ThreadableLoader.cpp`](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/Source/core/loader/ThreadableLoader.cpp),
teaches us two things.

1. Loaders are created by calling
[`ThreadableLoader::create`](https://code.google.com/p/chromium/codesearch#search/&q=ThreadableLoader::create&sq=package:chromium&type=cs)
or
[`ThreadableLoader::loadResourceSynchronously`](https://code.google.com/p/chromium/codesearch#search/&q=ThreadableLoader::loadResourceSynchronously&sq=package:chromium&type=cs).

2. That there are two concerete loader implementations,
`DocumentThreadableLoader` and `WorkerThreadableLoader`. The former is used for
the "normal" environment of a HTML document, and the latter is used for Web
Workers. The relevant headers and implementations are also in
[`Source/core/loader`](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/Source/core/loader/).


# Fetching

The data in each file (HTML, CSS, JavaScript, image, etc.) is loaded into a
`Resource`, which is stored in caches, and referenced by the DOM tree.
The resource interface is in
[`Source/core/fetch/Resource.h`](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/Source/core/fetch/Resource.h).



# References

* [Major objects in WebKit](http://www.webkit.org/coding/major-objects.html)
* [How WebKit loads a page](https://www.webkit.org/blog/1188/how-webkit-loads-a-web-page/)
