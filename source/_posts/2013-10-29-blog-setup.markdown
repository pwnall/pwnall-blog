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

I followed the official guides for
[Initial Setup](http://octopress.org/docs/setup/),
[Configuration](http://octopress.org/docs/configuring/), and
[Blogging](http://octopress.org/docs/blogging/). Although they are written for
Octopress 2.0, they still apply to 2.5.


# Heroku

The blog is deployed on the [Heroku platform](https://www.heroku.com/), and
using their generous free tier. I use one dyno and no plugins.

I did not follow the Octopress deployment guide for Heroku, because I did not
agree with the approach of checking in the generated static pages.

Instead, I used an
[Octopress buildpack](https://github.com/pwnall/heroku-buildpack-ruby),
which generates the static pages when I push to Heroku's git repository.

This build pack is basically
[Heroku's official Ruby buildpack](https://github.com/heroku/heroku-buildpack-ruby),
with the Octopress logic borrowed from
[jgarber's Octopress buildpack](https://github.com/jgarber/heroku-buildpack-ruby-octopress).
I'm betting that Heroku will update their buildpack much more frequently than
I'll have to update the Octopress bits, so I'm using a rebasing workflow for
the buildpack.

I followed
[jgarber's setup guide for the Octopress buildpack](http://jasongarber.com/blog/2012/01/10/deploying-octopress-to-heroku-with-a-custom-buildpack/),
with the exception of the Pygments setup, which is
[no longer required](http://jasongarber.com/blog/2012/01/10/deploying-octopress-to-heroku-with-a-custom-buildpack/#comment-750353224)

Also, since I'm using my own buildpack, the command for setting it up is
different from the command in the guide I followed.

```bash
heroku config:set BUILDPACK_URL=https://github.com/pwnall/heroku-buildpack-ruby.git#octopress
```


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
