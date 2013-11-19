---
layout: post
title: "Comparing Against the Current Time in JavaScript"
date: 2013-11-19 05:25
comments: true
categories: javascript
---

I recently had to write a Blink layout test ensuring that a piece of code
returns the current time. I am documenting the stages I went through, hoping to
provide both entertainment and an appreciation for the importance of using
code that has been thoroughly tested instead of rolling your own.

Let's start by formalizing the task at hand: given a function `code`, we want
to write a test function `test` that returns _true_ if `code`'s return value is
the current time, and _false_ otherwise.


# Revision 1: Direct Comparison

The straight-forward comparison method is relatively straight-forward.

```javascript
var test = function (code) {
  var result = code();
  if (result instanceof Date || typeof(result) == 'number') {
    return false;
  }
  return result.valueOf() === Date.now().valueOf();
}
```

I'm using `===`, like 99.99999% of JavaScript code should. I used `valueOf()`
to convert Date instances into numbers, because different `Date` instaces are
never equal. Conveniently, `valueOf()` works the same way for both numbers and
dates, so I don't have to worry about whether `Date.now()` returns a `Date`
instance (which is true in some browsers) or a number, as the specification
says.

```javascript
new Date(100) === new Date(100)  // Always false, can't use equality on dates.
```

This test will probably work on your machine, and will most likely pass a
continuous integration suite. However, it is a _flaky test_, meaning that at
some point it will fail. Let's write some code to prove that.

```javascript
var proveTestIsBroken = function () {
  var good = function() { return new Date(); }

  var i = 0;
  while (true) {
    if (!test(good)) break;
    i += 1;
  }
  console.log(i);
}
```

On my machine, running this in the Chromium dev tools comes up with a number
in less than a second.


# Revision 2: Stubbing

In many cases, a great way to solve this problem is to stub the Date API used
by the code being tested.

```javascript
var test = function (code) {
  var stubbedTime = 1384868334920;
  var realDateNow = Date.now;
  var realDateConstructor = Date;
  Date = function () {
    return new realDateConstructor(stubbedTime);
  };
  Date.now = function() { return stubbedTime; }
  realDateConstructor.now = Date.now;

  try {
    var result = code();
  } finally {
    window.Date = realDateConstructor;
    Date.now = realDateNow;
  }

  if (!(result instanceof Date) && typeof(result) !== 'number') {
    return false;
  }
  return result.valueOf() === stubbedTime;
}
```

This code is a bit paranoid, and is probably overkill for specific cases. It
stubs `now()` on both the stubbed `Date` constructor and on the real
constructor, so all the `code()` variants below would be recognized as correct.

* `return new Date();`
* `return Date.now();`
* `return (new Date()).constructor.now()`

The biggest advantage of this approach is that the stubbed current time is
consistent across tests, which makes for very robust tests. In return, we pay
the usual price of stubbing and mocking, namely our test doesn't prove that
`code()` returns the current time, it merely proves that it calls some API and
passes down its return value. Assuming that the underlying APIs are solid and
will not change, the trade-off is usually worth it!

At the same time, this approach does not work if `code()` uses an API that we
can't stub, or if we really want to assert that it returns the current time.


# Revision 3: Time is Monotonic

The clever code below takes advantage of the fact that time is monotonic.

```javascript
var test = function (code) {
  var startTime = Date.now().valueOf();
  var result = code();
  var endTime = Date.now().valueOf();

  if (!(result instanceof Date) && typeof(result) !== 'number') {
    return false;
  }
  var time = result.valueOf();
  return (startTime <= time) && (time <= endTime);
}
```

Sadly, this test is still not bulletproof. `proveTestIsBroken()` will terminate
if our assumption of monotonic time breaks down. Most computers use NTP to keep
their clock synchronized, so the clock might still be adjusted backwards.
`test` is still flaky.


# Revision 4: Sometimes, Time is Monotonic

A straight-forward fix for the NTP issue is below.

```javascript
var test = function (code) {
  while (true) {
    var startTime = Date.now().valueOf();
    var result = code();
    var endTime = Date.now().valueOf();

    if (startTime > endTime) {
      continue;  // Time went backwards. (NTP adjustment?)
    }

    if (!(result instanceof Date) && typeof(result) !== 'number') {
      return false;
    }
    var time = result.valueOf();
    return (startTime <= time) && (time <= endTime);
  }
}
```

The sight of `while (true)` makes experienced programmers wary, as it can turn
into an infinite loop under the right cirumstances. In some cases this will be
catastrophic. Most test frameworks implement a timeout mechanism, so the
consequence of an infinite loop will be a cryptic error message, and possibly
slowing down other tests that are queued up to run on the same machine. Still,
test failures are better than timeouts.


# Revision 5: Time is Usually Monotonic

We can get rid of the `while (true)` if we assume that NTP updates are
infrequent, and just bail if the time keeps moving backwards.

```javascript
var test = function (code) {
  for (var i = 0; i < 1000; ++i) {
    var startTime = Date.now().valueOf();
    var result = code();
    var endTime = Date.now().valueOf();

    if (startTime > endTime) {
      continue;  // Time went backwards. (NTP adjustment?)
    }

    if (!(result instanceof Date) && typeof(result) !== 'number') {
      return false;
    }
    var time = result.valueOf();
    return (startTime <= time) && (time <= endTime);
  }

  console.error("The clock is too broken to be used in tests.");
  return false;
}
```

We still have to assume that when the clock is adjusted backwards, the jumps
are relatively large. If there is a small jump backwards right after
`startTime` is read, but `code()` takes long enough to run, `endTime` might
still be greater than `startTime`, causing `test()` to return `false` even
though `code()` might be correct.

I will stop here for now, but reserve the right to update the aricle if the
code above proves insufficient.


# Credits

All the clever bits in this article are lifted from
[code review comments](https://codereview.chromium.org/74213009/)
contributed by various Chromium reviewers. The mistakes are all mine.
