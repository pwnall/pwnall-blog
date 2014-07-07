---
layout: post
title: "Blink's Public Interface"
date: 2014-07-07 02:10
comments: true
categories:
---

Blink is cleanly separated from Chromium's content layer. This article gives a
quick overview of Blink's interface. Most code references in this article are
relative to Blink's root directory,
[`third_party/WebKit`](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/).

This article is likely to get updated and grow as my understanding of the Blink
codebase improves.


## The Web and Platform APIs

The interface is defined by the files in the following directories:

* [`public/web/`](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/public/web)
  has headers for the classes implemented by Blink and used by the content
  layer; these classes are implemented in
  [`Source/web`](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/Source/web/)
* [`public/platform/`](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/public/platform)
  has classes implemented by the content layer and used by Blink; these classes
  are implemented in Chrome's content layer, in the
  [`content/`](https://code.google.com/p/chromium/codesearch#chromium/src/content/)
  directory in Chrome's source tree


Blink's public interface classes are all defined in the `blink` namespace, and
have the `Web` prefix in their name. In fact, when reading Blink code, it is
safe to assume that any class whose name starts with `Web` belongs to Blink's
public interface.


## Wrappers and Pointers

Headers in `public/` are not allowed to include files outside of the `public/`
directory. This implies that all the `blink::Web` classes and structures can
only use internal classes via pointers or references.

Many classes declared in `public/web` wrap Blink's internal classes. For
example,
[`public/web/WebNode.h`](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/public/web/WebNode.h)
defines `WebNode`, which wraps `WebCore::Node`, defined in
[`Sorce/core/dom/Node.h`](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/Source/core/dom/Node.h).

Some classes hold raw pointers to the internal classes that they wrap. For
example, `WebDatabase` (defined in
[`public/web/WebDatabase.h`](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/public/web/WebDatabase.h))
holds a raw pointer to the `WebCore::DatabaseBackendBase` instance that it
wraps. However, most classes use a smart pointer. For example, `WebNode` uses
the `WebPrivatePtr` smart pointer class to reference the `WebCore::Node`
instance that it wraps.

`WebPrivatePtr`, declared and completely defined in
[`public/platform/WebPrivatePtr.h`](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/public/platform/WebPrivatePtr.h),
is the most popular smart pointer class. It is used for wrapping
reference-counted classes, and automatically calls `ref()`, `deref()`, etc. on
the wrapped instance. The header has a very useful block comment right above
the `WebPrivatePtr` class declaration. On a first read, I'd skip the
boilerplate at the beginning of the file and go straight for the block comment.

`WebPrivateOwnPtr`, declared and completely defined in
[`public/platform/WebPrivateOwnPtr.h`](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/public/platform/WebPrivateOwnPtr.h),
is used for wrapping classes that are allocated on the heap, but do not use
reference-counting. The smart pointer automatically destructs the wrapped
instance. This class' header also has a very useful block comment.


## The Platform Object

The entry point to the implementations of the classes and functions defined in
the `public/platform` headers is the `blink::Platform` class, defined in
[`public/platform/Platform.h`](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/public/platform/Platform.h)
and implemented in
[`content/child/blink_platform_impl.h`](https://code.google.com/p/chromium/codesearch#chromium/src/content/child/blink_platform_impl.h)
in the Chromium souce tree.

Blink code obtains a reference to the current platform by calling the
`blink::Platform::current()` static function, and then calls a `Platform`
virtual method. Many methods are used to obtain instances of platform classes.

