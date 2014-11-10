---
layout: post
title: "Data buffers in Blink"
date: 2014-07-08 01:41
comments: true
categories: chromium blink
---

This article describes (some of) the classes used in Blink to represent data
buffers.

## WebData

`WebData` wraps a `SharedBuffer` instance. It is not thread-safe.

`WebData` is defined in
[public/platform/WebData.h](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/public/platform/WebData.h)
and implemented in
[Source/platform/exported/WebData.cpp](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/Source/platform/exported/WebData.cpp).

## WebThreadSafeData



