---
layout: post
title: "Iterating on Mobile Apps at Web Speed"
date: 2014-11-08T19:34:22-05:00
categories: web mobile cordova
---


This article covers a collection of techniques that allow me to build HTML5
applications with native capabilities, and iterate on my code using the
edit-save-refresh cycle that is characteristic of Web development.


## Motivation

I love building Web applications, and I'm addicted to the edit-save-refresh
cycle.

At the same time, I miss many features that mobile app developers take for
granted, but will take years to get through the Web standards gauntlet, such
as [WakeLock](http://www.w3.org/TR/wake-lock-use-cases/) and
[Push notifications](http://www.w3.org/TR/push-api/). Security decisions that
make perfect sense in the context of Web pages become really limiting for
applications. For example, it is impossible to build an alarm clock, because
the `play()` method on HTML5 media elements (`<audio>` and `<video>`) requires
user interaction, at least on
[Android](https://code.google.com/p/chromium/issues/detail?id=178297)
and
[iOS](https://developer.apple.com/library/safari/documentation/AudioVideo/Conceptual/Using_HTML5_Audio_Video/Device-SpecificConsiderations/Device-SpecificConsiderations.html#//apple_ref/doc/uid/TP40009523-CH5-SW4)

HTML5 application wrappers rose to respond to this challenge. A wrapper
essentially bundles a browser component (usually known as a `WebView`),
glue code that exposes native capabilities to the JavaScript inside the
WebView, and the HTML, CSS and JS code that you supply. The first wrapper that
I am aware of is [Titanium](https://github.com/appcelerator/titanium_mobile).

Today's favorite wrapper, [Apache Cordova](https://cordova.apache.org), has
made a lot of progress on the tooling front, but still uses the same bundling
technique, which means that I have to build a native application, deploy it on
my mobile device, and restart, whenever I want to make a change.


## Outline and Code

The following sections will describe the tricks I used to restore the
edit-save-refresh cycle inside a Cordova application container.

Some tricks in this article can be repurposed for other wrappers easily, while
some tricks are specific to Cordova's implementation. The latter may still
suggest approaches to solving similar problems in different wrappers.

I used the methods described here in the
[html5-app-container](https://github.com/pwnall/html5-app-container) project,
which is open-sourced on GitHub. The code samples in this article lack some of
the details in the GitHub project, in the interest of brevity and readability.
I consider the GitHub project code to be the production-quality version that
you should use for your applications.


## Chapter 1: The Naive Solution

In order to be able to use edit-save-refresh, I have to get the Cordova
container to load my application code from a server. The naive solution would
be to simply navigate to the server's URL from the application's main page.

```html www/index.html
<script>
window.location = 'http://webapp.com:3000/';
</script>
```

This is essentially the application code shipped with
[the initial html5-app-container version](https://github.com/pwnall/html5-app-container/tree/5a7dd32f04170441e12113d97a49d90f9cc91645). The key files are
[app/index.html](https://github.com/pwnall/html5-app-container/blob/5a7dd32f04170441e12113d97a49d90f9cc91645/app/index.html)
and
[app/js/index.js](https://github.com/pwnall/html5-app-container/blob/5a7dd32f04170441e12113d97a49d90f9cc91645/app/js/index.js).

While the application gets the edit-save-refresh cycle, it loses all the native
Cordova functionality. This is because the native functionality uses the
Cordova bridge, which is a fancy name for platform-specific hacks that let JavaScript code
communicate with the native Cordova code. The Cordova bridge is initialized by
loading a platform-specific `cordova.js` file that is placed along the HTML5
application's files by the Cordova build chain.

At this stage, it is tempting to attempt to load `cordova.js` from the Web
site. This could work on Android, where the HTML5 application is accessible
from `file:///android_asset/www/index.html`, but is doomed to fail on iOS,
because the application path has a GUID in it, e.g.
`file:///Users/pwnall/Library/Developer/CoreSimulator/Devices/CF0D2CC2-3E1B-49FB-BC59-AFE4FD4C65FD/data/Applications/75A1BEC5-6FBF-4E90-90B8-F4B8B22EF5B0/AppContainer.app/www/index.html`.

Even if the complex path on iOS can be tamed,  Chrome's remote inspector
reveals that accessing a `file:///android_asset` resource from a `http` origin
causes a security error, as shown below.

```html Web Application Snippet
<script src="file:///android_asset/www/cordova.js"></script>

<!-- produces the following security error
Not allowed to load local resource: file:///android_asset/www/cordova.js
-->
```

Before moving on to the next attempt, it's worth noting that even the naive
solution provides advantages over running the application from a Web browser.
Cordova's WebView uses a security model that is better suited for applications,
e.g. the `play()` method of HTML5 media elements can be called without user
interaction.

Furtermore, [the Crosswalk project](https://crosswalk-project.org/) offers a
Cordova fork that uses a WebView built on Chromium, instead of the platform's
native WebView. This approach removes a lot of variability coming from the
wide range of WebView versions shipped by device firmware. I can't recommend it
strongly enough! In fact, I wasted a few months trying to build
[pretty much the same thing](https://github.com/pwnall/chromeview), before this
existed.

Last, both Cordova and Crosswalk support debug and release builds. The debug
builds set up the WebViews so that they can be debugged remotely. On iOS, the
Cordova WebView can be debugged by
[Safari's Remote Inspector](http://webdesign.tutsplus.com/articles/quick-tip-using-web-inspector-to-debug-mobile-safari--webdesign-8787).
On Android, the Cordova WebView and the Crosswalk WebView can be debugged using
[Chrome's Remote Debugger](https://developer.chrome.com/devtools/docs/remote-debugging).

In both remote debuggers, hitting the Refresh keyboard shortcut (Command+R on
Mac OS X, Control+R on all other systems) reloads the WebView. This is an
important piece of the edit-save-refresh cycle, as having to reach out for the
phone and close/reopen the application would be much slower than hitting
Refresh on my development machine.


## Chapter 2: The Iframe

The `<iframe>` element has introduced many security issues in the Web platform,
so it seems fitting that we'd try to use it to work around Cordova's
restrictions.

A reasonable design would be to have the actual application running inside a
host page's `<iframe>`. The `<iframe>` takes up the host page's entire display
space. The host page is the default Cordova starting page,
`www/index.html`, so it can load `cordova.js` and talk to the Cordova
bridge.

```html www/index.html
<script src="cordova.js"></script>
<script src="js/index.js"></script>
<iframe id="iframe" src="blank.html" scrolling="no"></iframe>
<style>
  html, body, iframe {
    display: block;
    width: 100%; height: 100%;
    margin: 0; border: none; padding: 0;
  }
</style>
```

The host page uses
[postMessage](https://developer.mozilla.org/en-US/docs/Web/API/Window.postMessage)
to communicate with the application running inside the `<iframe>`.

```javascript www/www/index.js
(function() {
  var origin = 'http://webapp.com:3000';
  var appUrl = origin + '/';

  document.addEventListener('deviceready', function() {
    var iframe = document.getElementById('iframe');
    window.iframe = iframe;
    iframe.src = appUrl;
  }, false);
  window.addEventListener('message', function(event) {
    var iframe = document.getElementById('iframe');
    if (event.origin !== origin)
      return;
    if (event.data.substring(0, 25) === 'html5-app-container-eval|') {
      var javaScript = event.data.substring(25);
      var result = null;
      try {
        result = window.eval(javaScript);
      } catch(e) {
        result = e;
      }
      iframe.contentWindow.postMessage(
          'html5-app-container-evaled|' + result, '*');
    }
  }, false);
})();
```

To keep the initialization process simple, the host page first waits until the
Cordova bridge is initialized, and then loads the Web application in the
`<iframe>`.

The `postMessage` receiver only accepts messages whose origin matches the Web
application. This prevents hostile content, such as embedded ads, from
obtaining native capabilities.

The receiver shown above is simple by design. It essentially `eval()`s the
received message and responds with a message that contains the evaluation
result. Using `eval` means that I can iterate on all the interesting code by
changing the Web application code.

The main drawback of this approach is that sending JavaScript strings to the
host page is an unnatural technique for developing Web applications, and can
get tedious as the code that interacts with native features becomes more
complex. [Rails](http://rubyonrails.org/) (my Web framework of choice) does not
have good support for putting together JavaScript strings, and this was a
deal-breaker for me.

The second drawback is that debugging my application with Chrome or Safari's
remote inspector is a bit more tedious. In order to see my page's elements, I
have to peek into the host page's `<iframe>`. To play with the JavaScript
objects, I have to access the `<iframe>`'s `contentWindow`. The `deviceready`
handler code posted above assigins the `<iframe>` element to the `iframe`
property on the host page's window, to make debugging slightly easier.

Last, I was worried that as I use data-intensive features, such as
[Cordova's File API](http://plugins.cordova.io/#/package/org.apache.cordova.file),
shuttling large strings around might become a performance bottleneck.

Fortunately, we can do better!


## Chapter 3: The Iframe Grows Stronger

This section describes a trick that removes the need to `postMessage` strings
to the host page in most cases, by giving the Web application inside the
`<iframe>` direct access to Cordova's native objects. This makes the `<iframe>`
approach more bearable.

In a nutshell, after the Web application loads inside the `<iframe`>, it posts
a message to the host page, asking it to copy the references to Cordova's
objects to the `<iframe>`'s window object. For generality, the host page
creates an empty object, copies all the enumerable properties of its window
into the newly created object, then assigs the object to a property on the
`<iframe>`'s window.

```javascript www/js/index.js
(function() {
  var origin = 'http://webapp.com:3000';
  var appUrl = origin + '/';

  document.addEventListener('deviceready', function() {
    var iframe = document.getElementById('iframe');
    window.iframe = iframe;
    iframe.src = appUrl;
  }, false);
  window.addEventListener('message', function(event) {
    var iframe = document.getElementById('iframe');

    if (event.data === 'html5-app-container-wire-me') {
      var platform = {};
      iframe.contentWindow.cordovaPlatform = platform;
      var properties = Object.getOwnPropertyNames(window);
      var _length = properties.length;
      for(var index = 0; index < _length; ++index) {
        var name = properties[index];
        platform[name] = window[name];
      }
      iframe.contentWindow.postMessage('html5-app-container-wired-you', '*');
    }
  }, false);
})();
```

Note that the host page still waits for the Cordova bridge to be fully
initialized before navigating the `<iframe>` to the Web application. This
guarantees that the host page's window contains Cordova's native objects by
the time the `postMessage` receiver is invoked.

[The second major html5-app-container version](https://github.com/pwnall/html5-app-container/tree/9e53f5553d36dae4973bbd4dcbf463258cf3885e)
implements the trick above, combined with the `eval()` receiver described in
the previous section. The key files are
[app/index.html](https://github.com/pwnall/html5-app-container/blob/9e53f5553d36dae4973bbd4dcbf463258cf3885e/app/index.html)
and
[app/js/index.js](https://github.com/pwnall/html5-app-container/blob/9e53f5553d36dae4973bbd4dcbf463258cf3885e/app/js/index.js).

Unfortunately, this trick
[does not work in Crosswalk 10+](https://crosswalk-project.org/jira/browse/XWALK-2905),
so I had to look for a better solution.  However, it is worth knowing, as it
still works in Cordova, and might come in handy when dealing with other HTML5
application wrappers.


## Chapter 4: The Demise of the Iframe

A different approach to obtaining native capabilities is to attempt to
initialize the Cordova bridge in the Web application.

In this case, we don't embed any code in the Cordova application. Instead, we
add a [config.xml](http://cordova.apache.org/docs/en/4.0.0/config_ref_index.md.html)
whose `<access>` points to our Web application.

```xml assets/config.xml
<?xml version='1.0' encoding='utf-8'?>
<widget id="us.costan.app_container" version="1.0.0"
        xmlns="http://www.w3.org/ns/widgets"
        xmlns:cdv="http://cordova.apache.org/ns/1.0">
    <name>AppContainer</name>
    <description>
        Web Application Container
    </description>
    <author email="victor@costan.us" href="www.costan.us">
        Victor Costan
    </author>
    <content src="http://webapp.com:3000/" />
    <access origin="*" />
</widget>
```

According to [Cordova Bug 5988](https://issues.apache.org/jira/browse/CB-5988),
the Cordova bridge can be used from a `file://` origin, or from the top level
page's origin. So, if we can obtain the platform-specific JavaScript in the Web
application, we can execute it and gain access to Cordova's native features.

Fortunately, Cordova's JavaScript is well-structured, and collected in the
platform-specific assets directory for each platform. For example, the Android
JavaScript is all in `platforms/android/assets/www` and the iOS JavaScript is
in `platforms/ios/www`.

The first file we need is `cordova.js`, the build product of the
[cordova-js project](https://github.com/apache/cordova-js). As far as we're
concerned, it implements Cordova's module system (`cordova.define` and
`cordova.require`), and contains the platform-specific bootstrap code.

The second file we must extract is `cordova_plugins.js`, in the same directory.
It defines the `cordova/plugin_list` module, which exports a list of all the
plugins installed in the application.

The other files needed to set up the bridge are spread throughout the
`plugins/` directory. A reasonable method for obtaining them is recusively
looking for `.js` files inside the `plugins/` directory. In order to avoid
including any test code, we can filter out the files that don't start with
`cordova.define`.

We can collect and concatenate all the files for each platform into one big
bundle file, e.g. `android.js` for Android, and `ios.js` for iOS. In the Web
application, we need to detect whether we're running under Cordova and, if so,
load the appropriate bundle.

I use [the Rails asset pipeline](http://guides.rubyonrails.org/asset_pipeline.html)
to serve the JavaScript files, as it does all the tricks needed to serve them
with long-lived cache expiration dates. In order for this to work, I need to
use asset pipeline-generated URLs to load the scripts.

```javascript app/assets/javascripts/platform_scripts.js.erb
window.PlatformScripts = {
  android: <%= javascript_path('cordova/android.js').to_json %>,
  ios: <%= javascript_path('cordova/ios.js').to_json %>
};
```

The JavaScript bundles must not be executed when the Web application does not
run under Cordova. For example, the Android bundle currently uses
`window.prompt()` to talk to the native Cordova code. If this code runs outside
Cordova, the user will be presented with cryptic dialog boxes. In order to
avoid that, we must make sure that we run under Cordova before executing any
Cordova JavaScript.

```javascript app/assets/javascripts/platform.js
window.Platform = {};

// Returns "android", "ios", or null if not running under Cordova.
Platform.cordovaPlatform = function() {
  var ua = navigator.userAgent;
  if (/android/i.test(ua)) {
    if(window._cordovaNative)
      return 'android';
    return null;
  }
  if (/iphone|ipad|ipod/i.test(ua)) {
    if (\(\d+\)$/.test(ua))
      return 'ios';
    return null;
  }
  return null;
};
```

The code above takes advantage of Cordova implementation details, and was
derived from Cordova 3.6. On Android, Cordova defines a `_cordovaNative`
object. On iOS, Cordova appends a numeric nonce wrapped in parantheses to the
WebView's `User-Agent` property.

If the Web application detects that it runs under Cordova, a `<script>` tag
pointing to the appropriate script is dynamically generated and added to the
page.

```javascript app/assets/javascripts/platform.js
// Calls the callback with true (success) or false (failure).
Platform.bootCordova = function(callback) {
  var platform = Platform.cordovaPlatform();
  if (platform === null) {
    callback(false);
    return;
  }

  var element = document.createElement('script');
  element.src = PlatformScripts[platform];
  document.getElementsByTagName('head')[0].appendChild(element);

  document.addEventListener('deviceready', function() {
    callback(true);
  });
};
```

While implementing this approach, I ran over a
[Crosswalk crashing bug](https://crosswalk-project.org/jira/browse/XWALK-2907)
that brings down the entire Android application when a non-`file://` page
attempts to initialize the Cordova bridge. Fortunately, I was able to find and
submit a fix.

[The third major html5-app-container version](https://github.com/pwnall/html5-app-container/tree/4e0f66d0f805264bafbfe6e242aecc9ba8014ca1)
uses the approach outlined in this section. The only application file is
[app/config.xml](https://github.com/pwnall/html5-app-container/blob/4e0f66d0f805264bafbfe6e242aecc9ba8014ca1/app/config.xml).
The Cordova JavaScript specific to Android and iOS is collected by two bash
scripts,
[script/bundle-android.sh](https://github.com/pwnall/html5-app-container/blob/4e0f66d0f805264bafbfe6e242aecc9ba8014ca1/script/bundle-android.sh)
and
[script/bundle-ios.sh](https://github.com/pwnall/html5-app-container/blob/4e0f66d0f805264bafbfe6e242aecc9ba8014ca1/script/bundle-ios.sh).

This method addresses the main issues raised by the `<iframe>` approach. The
Web application does not need to `postMessage` JavaScript strings to a host
page, and the Chrome and Safari remote inspectors display the Web application's
elements and interact directly with the application's JavaScript world.

Unfortunately, this method's requirement of hosting Cordova's platform-specific
JavaScript bndles on the server introduces significant complexity in long-lived
production applications. For example, if a
[security issue](http://cordova.apache.org/announcements/2014/08/04/android-351.html)
is reported in the Cordova version used by the application wrappper, a new
version of the wrapper must be released. New bundles must be produced and
deployed on the production server before the new wrapper can be released.
Furthermore, the Web application must retain the bundles for all the wrapper
versions that were ever released.

From a performance standpoint, this approach has the downside of having to load
the platform-specific JavaScript over the network. The Android-specific
JavaScript file has almost 300kb, and the iOS-specific JavaScript is slightly
biggger. Furthermore, the file is loaded via a dynamically-added `<script>`
tag, which eludes
[the pre-loader](http://andydavies.me/blog/2013/10/22/how-the-browser-pre-loader-makes-pages-load-faster/).

The Rails asset pipeline sweetens the performance blow quite a bit, as the file
is 26kb after minifying and gzipping, and is served with a long-lived cache
expiration date. Still, it's nice to know that the following refinement removes
the performance issue completely, while being easier to maintain at the same
time.


## Chapter 5: The Return of the Iframe

In this section, we modify the approach presented above to remove the need for
server-hosted JavaScript bundles. The high-level idea is rather
straightforward, namely we must restructure our code so that the Web
application receives the JavaScript bundle from the Cordova wrapper. However,
we need a couple of tricks to bypass the security measures in Cordova's
WebView.

First, the build scripts for the Cordova-based wrapper must be augmented to
extract the JavaScript bundle, as described in the previous section.

This is mildly tricky, because the Cordova build scripts wipe the `www`
directory inside the platform-specific `platforms/` subdirectory at the
beginning of a build. Fortunately, the script copies the platform-specific
`cordova.js` from the `platform_www` directory inside the platform's
`platforms/` subdirectory, so we can stash our JavaScript bundle there.

Also, we minify the JavaScript bundle using
[UglifyJS 2](https://github.com/mishoo/UglifyJS2). On the Andoid and iOS
bundles, minification cut down about 300k of JavaScript to less than 100k. This
matters because we'll store the bundle in
[sessionStorage](https://developer.mozilla.org/en-US/docs/Web/Guide/API/DOM/Storage#sessionStorage),
which is read synchronously from JavaScript.

Second, we must send the JavaScript bundle to the Web application, despite the
fact that it cannot read `file://` resources. We bypass this limitation by
resurrecting the `<iframe>` technique discussed earlier.

The application wrapper includes a host page that loads a bootstrap page from
our Web application in a hidden `<iframe>`.

```javascript www/index.html
<iframe id="iframe" src="blank.html" style="display: none;"></iframe>
<script src="js/index.js"></script>
```

The bootstrap page receives the JS bundle from the host page via `postMessage`
and stores it in its `sessionStorage`. Note that the bootstrap page must be
hosted by the Web application, because the `sessionStorage` store is
per-origin. After the bundle is stored, the bootstrap page sends a message to
the host message telling it to navigate to the Web application's main page. The
main page URL is contained in the message so that the application wrapper only
needs to hard-code one URL, namely the bootstrap page's URL.

```javascript Sample Bootstrap Page
<script>
window.addEventListener('message', function(event) {
  if (event.origin.substring(0, 7) !== 'file://')
    return;
  var data = event.data;
  if (data.substring(0, 10) !== 'cordovaJs|')
    return;
  sessionStorage.setItem('cordova-js', data.substring(10));
  window.parent.postMessage('jump|http://webapp.com:3000/', '*');
});
if (window.parent !== window) {
  window.parent.postMessage('getjs', '*');
};
</script>
```

The host page JavaScript below looks large, but it can be de-constructed into
small straightforward parts. The `getBundle` function uses
[XMLHttpRequest](http://www.w3.org/TR/XMLHttpRequest/) to read the JavaScript
bundle. The `onMessage` function implements the `postMessage`-based protocol
used to communicate with the Web application's bootstrap page, and is similar
to the code for the `<iframe>`-based approach presented above.

```javascript www/js/index.js
(function() {
  var bootstrapUrl = 'http://webapp.com:3000/cordova';
  var bootstrapOrigin = 'http://webapp.com:3000';

  // Issues an XHR get and passes the response String to the callback function.
  var getUrl = function(url, callback) {
    var xhr = new XMLHttpRequest();
    xhr.open('GET', absoluteUrl, true);
    xhr.onreadystatechange = function() {
      if (xhr.readyState !== 4)
        return;
      var data = xhr.responseText || xhr.response;
      callback(data);
    };
    xhr.send();
  };

  // Loads the JS bundle via XHR and passes it to the callback function.
  var getBundle = function(callback) {
    var pageUrl = window.location.href;
    var slash = pageUrl.lastIndexOf('/');
    var absoluteUrl = pageUrl.substring(0, slash + 1) + 'cordova_all.min.js';
    getUrl(absoluteUrl, callback);
  };

  // Load the bootstrap URL in the iframe.
  var iframe = document.getElementById('iframe');
  iframe.src = bootstrapUrl;

  // The postMessage protocol used to talk to the bootstrap page.
  var onMessage = function(event) {
    if (event.origin !== bootstrapOrigin)
      return;
    var data = event.data;
    if (data === 'getjs') {
      getBundle(function(cordovaJs) {
        iframe.contentWindow.postMessage(
            'cordovaJs|' + cordovaJs, bootstrapOrigin);
      });
    }
    if (data.substring(0, 5) === 'jump|') {
      window.removeEventListener('message', onMessage);
      window.location = data.substring(5);
    }
  };
  window.addEventListener('message', onMessage, false);
})();
```

Note that after the JavaScript bundle transfer is complete, the host page's
code navigates the top-level window to the Web application's main URL. So,
after the bundle transfer, the Web application takes over the entire Cordova
WebView, as opposed to being contained inside an `<iframe>`, so the drawbacks
of the `<iframe>`-based solution do not apply.

This method also does not require the Web application to rely on Cordova
implementation details to detect whether it runs inside a Cordova container or
not. Instead, application code can simply probe `sessionStorage` for the
Cordova JavaScript bundle.

```javascript app/assets/javascripts/platform.js
// Calls the callback with true (success) or false (failure).
Platform.bootCordova = function(callback) {
  var cordovaJs = sessionStorage.getItem('cordova-js');
  if (!cordovaJs) {
    callback(false);
    return;
  }
  document.addEventListener('deviceready', function() {
    callback(true);
  });
  window.eval(cordovaJs);
};
```

This was implemented in
[the 4th major html5-app-container version](https://github.com/pwnall/html5-app-container/tree/4fe783bb58641fdebb87fd048fca0fdf2030863c)
uses the approach outlined in this section. The host page HTML is
[app/index.html](https://github.com/pwnall/html5-app-container/blob/4fe783bb58641fdebb87fd048fca0fdf2030863c/app/index.html)
and the loader code is in
[app/js/loader.js](https://github.com/pwnall/html5-app-container/blob/4fe783bb58641fdebb87fd048fca0fdf2030863c/app/js/loader.js).
The Cordova JavaScript bundles are collected by
[script/bundle-ios.sh](https://github.com/pwnall/html5-app-container/blob/4fe783bb58641fdebb87fd048fca0fdf2030863c/script/build-ios.sh),
which (confusingly) builds the Cordova-based wrapper for both iOS and Android.
[script/bundle-android.sh](https://github.com/pwnall/html5-app-container/blob/4fe783bb58641fdebb87fd048fca0fdf2030863c/script/build-android.sh)
builds an Android wrapper using the Crosswalk fork of Cordova.

Unfortunately, the code here has an XSS (cross-site scripting) vulnerability.
If an attacker convinces a user to download a malicious HTML page and open the
downloaded version, such that it gets the `file://` origin, the malicious page
can open our Web application's bootstrap page in an `<iframe>` and use the
`postMessage` protocol to convince the bootstrap page to store some evil
JavaScript in `sessionStorage`. If the user then visits our Web application
from the same computer, the application will run the attacker's evil
JavaScript.


## Chapter 6: The Revenge of the Iframe

In order to fix the XSS vulnerability, we apply a well-known method for
preventing against CSRF (cross-site request forgery) vulnerabilities.

The Web application's bootstrap page generates a random token and places it
inside a `<div>` attribute in its DOM tree. The application wrapper uses its
privileges to extract the token from the `<div>` element, and sends the token
together with the Cordova JavaScript. The bootstrap does not write to
`sessionStore` if the token in the message doesn't match the value that it
generated.

The main source of complexity in the updated bootstrap page code (shown below)
is the token generation code. We first attempt to generate a cryptographically
secure token, using the [WebCrypto API](http://www.w3.org/TR/WebCryptoAPI/).
If that is not supported, we fall back to using `Math.random()`.

```javascript Sample Bootstrap Page
<div id="cordova-js-token" data-token=""></div>
<script>
(function() {
  var generateToken = function() {
    var numbers = null;

    // If possible, get cryptographically secure random values.
    try {
      if (window.crypto && crypto.getRandomValues && window.Uint32Array) {
        numbers = new Uint32Array(4);
        crypto.getRandomValues(numbers);
      }
    } catch(cryptoError) {
      numbers = null;
    }

    // Fallback to weak random values.
    if (numbers === null) {
      numbers = new Array(4);
      for (var i = 0; i < 4; ++i)
        numbers[i] = Math.random();
    }

    var strings = new Array(4);
    for (var i = 0; i < 4; ++i)
      strings[i] = numbers[i].toString(36);
    return strings.join('_');
  };
  var jsToken = generateToken();

  var checkToken = function(submitted, baseline) {
    return submitted === baseline;
  };

  var onMessage = function(event) {
    if (event.origin.substring(0, 7) !== 'file://')
      return;
    var data = event.data;
    if (data.substring(0, 10) !== 'cordovaJs|')
      return;
    var tokenEnd = data.indexOf('|', 10);
    if (tokenEnd === -1)
      return;
    if (checkToken(data.substring(10, tokenEnd), jsToken))
      sessionStorage.setItem('cordova-js', data.substring(tokenEnd + 1));

    window.parent.postMessage('jump|http://webapp.com:3000/', '*');
  };

  window.addEventListener('message', onMessage);
  var tokenDiv = document.getElementById('cordova-js-token');
  tokenDiv.setAttribute('data-token', jsToken);
  window.parent.postMessage('getjs', '*');
})();
</script>
```

Note that when the code above receives an incorrect token, it follows through
the `postMessage` protocol, but doesn't update `sessionStorage`. This reduces
the amount of information that a potential attacker might obtain, at the cost
of making it harder to debug the JavaScript bundle transmission process.

The host page's code, embedded in the application wrapper, is identical to the
previous version, up to the `getIframeToken` function. The main source of
complexity here is working around browser support for the `contentDocument`
property. The `onMessage` function was modified to call out to
`getIframeToken` and embed the return value in the message that it posts to
the bootstrap page.

```javascript www/js/index.js
(function() {
  var bootstrapUrl = 'http://webapp.com:3000/cordova';
  var bootstrapOrigin = 'http://webapp.com:3000';

  // Issues an XHR get and passes the response String to the callback function.
  var getUrl = function(url, callback) {
    var xhr = new XMLHttpRequest();
    xhr.open('GET', absoluteUrl, true);
    xhr.onreadystatechange = function() {
      if (xhr.readyState !== 4)
        return;
      var data = xhr.responseText || xhr.response;
      callback(data);
    };
    xhr.send();
  };

  // Loads the JS bundle via XHR and passes it to the callback function.
  var getBundle = function(callback) {
    var pageUrl = window.location.href;
    var slash = pageUrl.lastIndexOf('/');
    var absoluteUrl = pageUrl.substring(0, slash + 1) + 'cordova_all.min.js';
    getUrl(absoluteUrl, callback);
  };

  // Load the bootstrap URL in the iframe.
  var iframe = document.getElementById('iframe');
  iframe.src = bootstrapUrl;

  // Returns the HTMLDocument inside an <iframe>, or null.
  var getIframeDocument = function(iframe) {
    return iframe.contentDocument || iframe.contentWindow.document;
  };

  // Gets the XSS protection token in the <iframe>.
  var getIframeToken = function(iframe) {
    var iframeDoc = getIframeDocument(iframe);
    var tokenDiv = iframeDoc.getElementById('cordova-js-token');
    return tokenDiv.getAttribute('data-token');
  };

  // The postMessage protocol used to talk to the bootstrap page.
  var onMessage = function(event) {
    if (event.origin !== bootstrapOrigin)
      return;
    var data = event.data;
    if (data === 'getjs') {
      getBundle(function(cordovaJs) {
        iframe.contentWindow.postMessage(
            'cordovaJs|' + cordovaJs, bootstrapOrigin);
      });
    }
    if (data.substring(0, 5) === 'jump|') {
      window.removeEventListener('message', onMessage);
      window.location = data.substring(5);
    }
  };
  window.addEventListener('message', onMessage, false);
})();
```

The security in this method can be proven by reasoning that an attacker who can
extract the token from the bootstrap page can also extract a CSRF token
embedded in a `<form>`, `<input>` or `<meta>` tag. Such an attacker can already
execute requests on behalf of its victim.

Unfortunately, just like the `contentWindow` trick described in a previous
section, the code above does not work in Crosswalk 10, because of
[this bug](https://crosswalk-project.org/jira/browse/XWALK-2905).

If you want to use Crosswalk versions impacted by the bug, the host page
JavaScript must be modified to catch security exceptions and return a special
CSRF token, such as `0000`. The bootstrap page can accept the special token if
it can verify that the JavaScript execution environment belongs to an
impacted Crosswalk version.

```javascript Host Page Changes
var getIframeDocument = function(iframe) {
  try {
    if (iframe.contentDocument)
      return iframe.contentDocument;
  } catch(securityError) {
  }
  try {
    if (iframe.contentWindow && iframe.contentWindow.document)
      return iframe.contentWindow.document;
  } catch(securityError) {
  }
  return null;
};

var getIframeToken = function(iframe) {
  var iframeDoc = getIframeDocument(iframe);
  if (iframeDoc === null)
    return '0000';

  var tokenDiv = iframeDoc.getElementById('cordova-js-token');
  return tokenDiv.getAttribute('data-token');
};
```

The host page changes above make the token extraction code more robust, and
have no negative impact, from a security perspective. Therefore, they are
included in the `html5-app-container` code.

The Web application page gets to decide whether it accepts the special CSRF
token. Therefore the modification below can simply be skipped by an application
that never releases a wrapper based on a Crossswalk version impacted by the bug
above. Although the changes

```javascript Crosswalk Bootstrap Page Changes
var checkToken = function(baseline, submitted) {
  if (submitted === baseline)
    return true;

  // Work around XWALK-2905.
  if (submitted !== '0000')
    return false;
  if (!window._cordovaNative)
    return false;
  if (_cordovaNative.exec.toString() !== 'function () { [native code] }')
    return false
  if (navigator.userAgent.indexOf('Crosswalk/10.') === -1)
    return false;
  return true;
};
```

This was implemented in
[the 5th major html5-app-container version](https://github.com/pwnall/html5-app-container/tree/35509c975b963e23783ea20f197782ba97dcf767)
uses the approach outlined in this section. The new loader code is in
[app/js/loader.js](https://github.com/pwnall/html5-app-container/blob/35509c975b963e23783ea20f197782ba97dcf767/app/js/loader.js).

The code presented above has a couple of performance issues, compared to a
wrapper that loads the Web application directly into a WebView.
Time-permitting, they may be the subject of future work.

First, loading the bootstrap page adds an HTTP round-trip to the application's
startup time. This can be significant for applications that are meant to be
used on the go. The round-trip can be eliminated from most starts by serving
the bootstrap page with a far-in-the-future cache expiration time.

A high-performance solution would remove the bootstrap page completely.
Instead, a
[Cordova plugin](http://cordova.apache.org/docs/en/4.0.0/guide_hybrid_plugins_index.md.html)
would serve the JavaScript bundle from a custom URL scheme. The Web application
could attempt to load the bundle via an XMLHttpRequest, and would report a
Cordova boot failure if the request fails. This approach would be expensive
in terms of engineering time, as a Cordova plugin requires different native
code for each supported platform, and must be maintained to track updates to
the Cordova API.

Second, storing the (rather large) JavaScript bundle in `sessionStorage` is a
source of performance issues. The `sessionStorage` API consists of blocking
calls, so the WebView's JS thread is unresponsive while the mobile device reads
or writes the JavaScript bundle into the `sessionStorage` backing store. Also,
some WebView implementations store the `sessionStorage` contents in RAM, so the
JavaScript bundle might use 200kb of RAM (the bundle is 100kb after
minification, but is likely stored as a
[UTF-16 or UCS-2 string](http://stackoverflow.com/questions/8715980/javascript-strings-utf-16-vs-ucs-2).
So, we're possibly taking up 200kb of RAM with a string that's only used once
during page load.

The `storageSession` is used in these samples because of its simplicity and
wide support. A performance-conscious Web application could use
[IndexedDB](https://dvcs.w3.org/hg/IndexedDB/raw-file/tip/Overview.html) and
fall back to the deprecated
[Filesystem API](http://www.w3.org/TR/file-system-api/) and
[WebSQL database API](http://www.w3.org/TR/webdatabase/).
Unfortunately, all fallbacks are necessary to achieve reasonable platform
coverage. On a brighter note, the choice of bundle backing store is completely
decoupled from the code inside the Cordova wrapper, so a Web application can
deploy this performance improvement without having to change the code in
`html5-app-container`.

Of course, the problem of storing the JavaScript bundle goes away if the
bootstrap page is eliminated by implementing a Cordova plugin.


## JavaScript Bundle Signatures?

A noteworthy alternative to the CSRF token extraction method described here is
signing the JavaScript bundles. The signatures would be produced during the
application wrapper build process, and verified by the code in the bootstrap
page.

Unfortunately, this leaves secret key signing methods, like HMAC, out of
the question, because the signature would be embedded on the bootstrap page.
Verifying asymmetric (e.g., RSA) signatures is slow when done in JavaScript.
Furthermore, signing would burden the build process with the extra complexity
of handling a secret key.

Last but not least, relying on signatures would mean that the bootstrap page
would accept any previously signed JavaScript bundle. A troll could use this to
load a Cordova bundle into the site when used from a normal browser, which
would cause JavaScript prompts to pop up when the user browses to our Web
application.

A more serious attacker could load an old (but still signed) version of a
bundle with known security vulnerabilities, and exploit the vulnerabilities
against the site.


## General Security Considerations

Unfortunately, all the designs above have the drawback that any XSS (cross-site
scripting) vulnerability can be used to obtain access to native features.

At the same time, it is worth mentioning that native applications also suffer
from security vulnerabilities, such as
[push notifications spam and hijacking](https://www.usenix.org/system/files/conference/woot12/woot12-final7.pdf)
and
[Android intent hijacking](http://css.csail.mit.edu/6.858/2012/projects/ocderby-dennisw-kcasteel.pdf).
Furthermore, a security patch in a Web application requires a server push to
be deployed, which can be done in minutes or at most hours. A mobile
application patch requires an app store / market submission, which often
includes a human review, and can take between hours and days.


## Conclusion

This article described the tricks I used in the
[html5-app-container](https://github.com/pwnall/html5-app-container) project,
which allows me to build Web applications with the native capabilities afforded
by [Cordova](http://cordova.apache.org/), and iterate on my code using the
edit-save-reload cycle that is typical of Web applications.
