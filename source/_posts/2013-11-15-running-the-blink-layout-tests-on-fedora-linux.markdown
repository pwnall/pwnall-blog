---
layout: post
title: "Running the Blink Layout Tests on Fedora Linux"
date: 2013-11-15 11:02
comments: true
categories: chromium blink fedora
---

I use Fedora instead of the recommended Ubuntu distribution. My main motivation
is that Fedora generally ships newer packages than Ubuntu, so I get to try out
newer versions of software earlier than most people. In the context of Chromium
and Blink development, this means I'm more likely to catch bugs like
[this one](https://codereview.chromium.org/26432002/).

This is a list of the steps I did to be able to run Blink's layout tests on
Fedora Linux. Layout tests are currently main method of testing in Blink, so
being able to run them is a pretty hard prerequisite to working on patches.


# Build Prerequisites

The
[Fedora Setup section](https://code.google.com/p/chromium/wiki/LinuxBuildInstructionsPrerequisites#Fedora_Setup)
of the Linux build prerequisites in the Chromium wiki has two commands that
can be conveniently copy-pasted to get most of the packages needed for building
Chromium. The optional packages are needed for running Blink's layout tests, so
make sure you run both commands.

The comments section also has a list of packages that should be installed, and
somewhat makes up for the fact that the Chromium codebase evolves much faster
than the Wiki page.

After installing all the packages, you should be able to build the
`blink_tests` target, which contains the Content Shell and all the tools used
to run the layout tests.

```bash
cd ~/chromium/src
ninja -C out/Debug blink_tests
```


# Fonts for the Layout Tests

The layout tests refuse to run without the fonts used by the pixel tests. The
font list and lookup details are in
[`src/content/shell/app/webkit_test_platform_support_linux.cc`](https://code.google.com/p/chromium/codesearch#chromium/src/content/shell/app/webkit_test_platform_support_linux.cc).
If the path above changes, try
[searching the codebase for kochi-gothic.ttf](https://code.google.com/p/chromium/codesearch#search/&q=kochi-gothic.ttf&sq=package:chromium&type=cs).

The easiest method for obtaining the needed fonts is:

1. [Download an Ubuntu desktop ISO](http://www.ubuntu.com/download/desktop).

2. Use [VirtualBox](https://www.virtualbox.org/) (or your favorite alternative)
   to set up an Ubuntu VM.

3. Open up the
   [Ubuntu Setup section](https://code.google.com/p/chromium/wiki/LinuxBuildInstructionsPrerequisites#Ubuntu_Setup)
   in a browser in the VM.

4. Install the font-related packages (look for _font_ or _ttf_ in the package
   name) in the VM.

5. Copy the entire `/usr/share/fonts/truetype` directory to your Fedora
   installation.

    ```
    mkdir ~/Downloads/ChromiumFonts
    scp -r you@vmname.local:/usr/share/fonts/truetype ~/Downloads/ChromiumFonts
    sudo cp -r ~/Downloads/Chromiumfonts/truetype /usr/share/fonts
    ```

6. Check that the layout tests are happy with your font setup.

    ```
    cd ~/chromium/src
    out/Debug/content_shell --dump-render-tree ~/chromium/src/third_party/WebKit/LayoutTests/fast/js/array-indexof.html
    ```

7. Stash the fonts in your Dropbox or Google Drive, so you won't have to do
   this again.

8. Power off and remove the Ubuntu VM.

For whatever it's worth, the Microsoft fonts can be easily obtained by
installing the RPM from the
[mscorefonts2 project](http://sourceforge.net/projects/mscorefonts2/). However,
a reasonably thorough search for `koichi-gothic.ttf` led to no results, so the
Ubuntu VM method is the fastest method I know for getting all the fonts.


## Story: Debugging a Crash in the Layout Tests Crash

On one occasion, all the layout tests that I ran resulted in crashes. The steps
I took might be helpful for shedding light on a similar situation.

First, I first found an "easy" test that really shouldn't crash the renderer,
so I wouldn't have to worry about whether the crash was introduced by a recent
change. My test was
[`fast/js/array-indexof.html`](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/LayoutTests/fast/js/array-indexof.html),
but other tests in that directory might be equally suitable.

Running the test with the extra `--driver-logging` option revealed the issue.
(I was missing fonts.)

```bash
webkit/tools/layout_tests/run_webkit_tests.sh --debug --driver-logging fast/js/array-indexof.html
```

Had that not worked, I would have ran the test in the Content Shell directly.
Note that unlike the full Chromium build, the Content Shell can't convert
relative paths into URLs, so I'm giving it a full path. I'm relying on bash to
expand `~` to my home directory.

```bash
out/Debug/content_shell ~/chromium/src/third_party/WebKit/LayoutTests/fast/js/array-indexof.html
```

In my case, this command worked, and showed me that the problem was somewhere
in the testing infrastructure, so I ran the test with the `--dump-render-tree`
flag.

```bash
out/Debug/content_shell --dump-render-tree ~/chromium/src/third_party/WebKit/LayoutTests/fast/js/array-indexof.html
```
