---
layout: post
title: "Smart pointer types in Blink"
date: 2013-11-10 01:47
comments: true
categories: chromium blink
published: false
---

Raw pointers show up rarely in Blink code. Instead, most of the code uses the
smart pointer types
[`RefPtr`](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/Source/wtf/RefPtr.h),
[`PassRefPtr`](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/Source/wtf/PassRefPtr.h),
[`OwnPtr`](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/Source/wtf/OwnPtr.h) and
[`PassOwnPtr`](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/Source/wtf/PassOwnPtr.h).
I think that figuring out how to properly use each type was harder than it
needs to be, so I set out to document my findings.


# Smart Pointers Help Resource Management

Blink can't afford automated memory management (e.g. garbage collection), and
instead


