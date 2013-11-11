---
layout: post
title: "Searching the Chromium Codebase"
date: 2013-11-09 17:06
comments: true
categories: chromium
---

After I started programming professionally, I was literally shocked when I
asked my tech lead "where is X implemented?", and his answer was "let's `grep`
for it". My schoolwork consisted of writing a new piece of software, or
completely understanding a piece of software, and then using or modifying it.
At the same time, if I wanted to look up a piece of information, I would use
Google instead of going to a library and trying to figure out what book might
hold my answer.

Long story short, `grep` is a central piece to finding one's way in a large
codebase, just like Google is the only sensible way of finding specific bits of
information in the large pool of human knowledge that we have on the Web.


# Git Grep

`git grep` works for any repository, so it was go-to tool when I started
hacking on the Chromium codebase. I still use it when I'm traveling and I don't
have Internet access. For most cases, Google's code search (covered in the
following section) is a better tool.

`git grep` is better than the plain vanilla `grep` because it searches over the
entire repository. However, Chromium uses many repositories, so `git grep` does
not cover the entire codebase.

For example, the search below does not cover the Blink repository.

```bash
cd ~/chromium/src
git grep -n -3 allowImage
```

Moving to Blink's root directory will make `git grep` search the Blink
repository. For this reason, I usually have at least one Terminal tab open in
`~/chromium/src` and one tab open in `~/chromium/src/third_party/WebKit`.

```bash
cd ~/chromium/src/third_party/WebKit
git grep -n -3 allowImage
```

I found the command-line arguments above to be generally useful. `-n` shows
line numbers, `-3` provides 3 lines of context above and below the match. 3 is
not a magic value, all digits work similarly.

Bash assigns special meaning to some characters. Remember to escape them or to
quote the pattern when necessary.

```bash
cd ~/chromium/src/third_party/WebKit
git grep -n -3 "allowImage("
git grep -n -3 allowImage\\(
```

If you find yourself using `git grep` a lot (presumably for other projects),
consider upgrading to [ack](http://beyondgrep.com/).


# Code Search: Grep on Steroids

Google's
[Code Search service](http://en.wikipedia.org/wiki/Google_Code_Search) was shut
down in 2013, but it is still up and running for the Chromium code base, at
[cs.chromium.org](http://cs.chromium.org).

The main advantage of Chromium Code Search is the search speed. For example,
`time git --no-pager grep -n -3 "allowImage("` took 1.9 seconds in the main
Chromium repository and 2.7 seconds in the Blink repository, on a high-end
mid-2012 Retina MacBook Pro. Chromium Code search answered the same query for
_the entire Chromium codebase_ in 0.2 seconds.

Here are the three features that I didn't discover right away, but I used a
lot once I figured them out.

1. Hovering over a name turns it into a link to the definition for the
   respective variable, function, method or class. My Chromium setup opens a
   link in a new tab when I click on the middle mouse button, and I use this a
   lot when exploring Chromium code.

2. Clicking over a name where it is defined opens up the XRefs
   (cross-references) pane, which is much better than a raw search for commonly
   used names.

3. The file tree on the left can be replaced with an outline of the current
   file. Look for the _Outline_ link above the tree.

If you'd like to use code search for other projects, you might be interested in
the
[open-sourced Google Code Search back-end code](https://code.google.com/p/codesearch/).
Unfortunately, this is not the full code for the Web application, but rather a
starting point. Even if you don't end up using the implementation there, the
comments in the code are very insightful.


# Active Exploration

This post covered passively exploring the Chromium codebase by searching.
Sometimes, passive search is inefficient, and must be complemented by active
exploration methods such as
[logging]({% post_url 2013-10-25-logging-in-chromium %}) and
getting stack traces (upcoming post).


# References

* [Man page for git grep](https://www.kernel.org/pub/software/scm/git/docs/git-grep.html)
* [Top 10 reasons to use ack for source code](http://beyondgrep.com/why-ack/)
* [Regular Expression Matching with a Trigram Index or How Google Code Search Worked](http://swtch.com/~rsc/regexp/regexp4.html)
