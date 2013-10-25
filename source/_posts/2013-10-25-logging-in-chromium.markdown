---
layout: post
title: "Logging in Chromium"
date: 2013-10-25 04:39
comments: true
categories:
---

To me, *logging* is a fancy way of saying `printf`-debugging. In a large
project, like Chromium, it comes in very handy for learning about the code.
Strategically-placed logging statements can help you prove that a function is
called in response to an action, or figure out the values that some parameters
take.

Chromium puts together many different projects, and they all have their logging
subsystems. I will cover logging for the projects that I worked with. That
being said, I'm a people pleaser, so I am likely to add a project, if you ask
nicely.


# Chromium

The Chromium source code follows Google's coding style, and the logging API
resembles the API used in Google's internal codebases. In particular, most
logging statements are compiled in both the release and the debugging builds,
and can be activated via command-line flags.

To enable Chromium logging, run your build with the following arguments.

```bash
path/to/chromium --enable-logging=stderr --v=1
```

Logging statemtents look like using streams in the C++ standard library.

```c++
#include "base/logging.h"

VLOG(1) << "SomeFunction(" << arg << ")";
```

Using `DVLOG` instead of `VLOG` causes a logging statement not to be compiled
into release builds. You shouldn't worry about this while you use logging for
exploratoraty purposes. When you get in a position to commit a logging
statement, your code reviewer can help you figure out the specific logging
statement that you should use.


# Blink

The Blink source code in `/third_party/WebKit` uses Apple-style logging.
Most importantly, logging statements are only compiled in debugbuilds.
Unfortunately, this means that if you need logging, you can't use release
builds, which are faster to produce.

To enable Blink logging, run your (debug) build with the following arguments.

```bash
path/to/chromium --webcore-log-channels=Loading,ResourceLoading
```

The valid channels are defined in `Logging.cpp` in the WebKit source tree. The
file has moved in the recent past, so the command below is a good way of
finding it.

```bash
cd ~/chromium/src/third_party/WebKit
find . | grep Logging.cpp
```

Logging statements look like `printf` calls.

```c++
#include "platform/Logging.h"

LOG(Loading, "ResourceFetcher::requestResource %s", url.latin1().data());
```


# V8

V8 can run on its own, so you should try to do your development that way
whenever possible.

To enable V8 logging, run your build with the following arguments.

```bash
path/to/chromium --js-flags="--log_all --logfile=-"
```

Running a tool such as `tick-processor` requires that V8's logging output is
neatly separated from Chromium's. To have V8 output its log to a file,
Chromium's sandbox must be disabled. By default, V8 logs to `v8.log`.

```bash
path/to/chromium --no-sandbox --js-flags="--log_all"
```

I haven't hacked on the V8 source code yet, so I don't have an example of
writing logging statements.


# References

* [Run Chromium with flags](http://www.chromium.org/developers/how-tos/run-chromium-with-flags)
* [How to enable logging](http://www.chromium.org/for-testers/enable-logging)
* [CL: Expose WebCore log channels on the Chrome command line](https://codereview.chromium.org/6528016/)
* [Profiling Chromium with V8](https://code.google.com/p/v8/wiki/ProfilingChromiumWithV8)
