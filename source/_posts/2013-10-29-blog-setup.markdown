---
layout: post
title: "Blog Setup"
date: 2013-10-29 17:19
comments: true
categories: meta
---

This article is a quick description of the software stack used to publish this
blog.


# Octopress

The blog uses the [2.5 branch](https://github.com/imathis/octopress/tree/2.5)
of the [Octopress static blog generator](http://octopress.org/).

I have set up the Octopress repository as a remote called `up` (upstream), and
my `2.5` branch mirrors the matching Octopress branch. This means that I can
adopt updates with the following commands.

```bash
git checkout 2.5
git pull up 2.5
git checkout master
git rebase 2.5
```


# Heroku

The blog is deployed on the [Heroku platform](https://www.heroku.com/), and
using their generous free tier. I use one dyno and no plugins.

I followed the official guides for
[Initial Setup](http://octopress.org/docs/setup/),
[Configuration](http://octopress.org/docs/configuring/), and
[Blogging](http://octopress.org/docs/blogging/). I did not follow the Heroku
deployment guide, because I did not agree with the approach of checking in the
generated static pages.

Instead, I used
[jgarber's Octopress buildpack](https://github.com/jgarber/heroku-buildpack-ruby-octopress),
which generates the static pages when I push to Heroku.
I followed the buildpack's
[official setup guide](http://jasongarber.com/blog/2012/01/10/deploying-octopress-to-heroku-with-a-custom-buildpack/),
with the exception of the Pygments setup, which is
[no longer required](http://jasongarber.com/blog/2012/01/10/deploying-octopress-to-heroku-with-a-custom-buildpack/#comment-750353224)


# Discussion

I really like that I have my contents in a portable form (Markdown), so I can
migrate to a different blogging system, such as [ghost](https://ghost.org),
if I want to.

I don't really like the Octopress theme. I like the styling in
[ghost](https://ghost.org/features/) and the typography in
[medium](https://medium.com/). Octopress won because of its simple setup
(Ghost stores images in the filesystem, so it doesn't work well on Heroku),
because it's not restrictive (medium requires draft reviewers to log in with
Twitter). Good support for code highlighting doesn't hurt.

Given an unlimited amount of time, I would have used the
[middleman static site generator](http://middlemanapp.com/) with its
[support for blogs](http://middlemanapp.com/blogging/) and
[syntax highlighting](https://github.com/middleman/middleman-syntax).
