---
layout: post
title: "Build Your Own Chromium"
date: 2013-10-24 01:04
comments: true
categories: chromium
---

I initially planned to start this series with some introductory material, such
as giving a tour of the project, or explaining why I think contributing is a
worthy endeavor.

At the same time, setting up Chromium will keep your development machine busy
for a long time. Start this right away, so you won't be blocked waiting for
your computer later on.


# Prerequisites

Chromium development requires a reasonably powerful machine. Your computer
should have at least 4GB of RAM and at least 60GB of free disk space.

This article contains an easy way of getting a Chromium development
environment. For the sake of simplicity, I made some decisions for you, and
didn't waste space documenting alternatives.

You can make different choices, and you're responsible for figuring out how
they impact my instructions. In general, you can get away with any change, as
long as you can read the full Chromium guides, and use Google and StackOverflow
to figure out error messages.


# Use a UNIX Operating System

My articles assume you use a UNIX operating system. You'll have the easiest
time developing on OS X, but running it requires an Apple computer. For PCs,
the best environment for Chromium development is the
[Ubuntu Linux distribution](http://www.ubuntu.com/download).

Mainstream Linux distributions, such as Fedora and Arch are also supported, but
you'll have to do a bit more research and debugging to if some of the commands
don't work right away.

Windows is not suitable for Chrome development, and I recommend against using
it. The only good reason for running Windows on a machine is hardware support.
If your machine needs Windows drivers, you should use a virtualization product,
such as [VirtualBox](https://www.virtualbox.org/wiki/Downloads) or
[VMWare Player](http://www.vmware.com/go/downloadplayer/), and
[get Ubuntu Linux](http://www.ubuntu.com/download). You should allocate at
least 4GB of RAM and 64 GB of disk space to the VM that you use to run Ubuntu.


# Install Git

You need git to be able to contribute to many popular open-source software. If
you don't know how to use git, worry about that later. Remember we're trying to
kick off all the downloads you need to hack on Chromium, so you can read up on
stuff you need while your dev machine downloads code.

On OS X, installing [Apple's XCode](https://developer.apple.com/xcode/) will
give you git and all the other tools you need to build Chromium.

On Ubuntu, run the command below in *Terminal*. Other Linux distributions
require similar commands.

```bash
sudo apt-get install git
```


# Install depot_tools

Chromium uses code from dozens (if not hundreds) of code repositories. The only
sane way of getting all the code is to use depot_tools, which is their
repository management software.

Copy-paste the commands below in *Terminal*.

```bash
cd ~
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
```


# Set Your Environment Variables

The Chromium tools use a few environment variables. To make your life easy,
they should be always set on your development machine. The best way to achieve
that is to add the lines below to `~/.bash_profile` (on OS X) or to `~/.bashrc`
(on Linux).

```bash
export GYP_GENERATORS=ninja
export PATH=$PATH:$HOME/depot_tools
export CHROME_DEVEL_SANDBOX=$HOME/chromium/src/out/Debug/chrome_sandbox
```

On OS X, the easiest way of editing the file I mentioned is to run the commands
below in *Terminal*, and adding the lines above in the *TextEdit* window that
shows up.

```bash
touch ~/.bash_profile
open ~/.bash_profile
```

On Ubuntu Linux, use the commands below instead.

```bash
touch ~/.bashrc
gedit ~/.bashrc
```


# Get the Chromium Source Code

You are now ready to download the all source code used to build Chromium. If
you have a laptop, go to the place with the best Internet connection that you
have access to, because you'll be downloading a few gigabytes of code.

Run the commands below in *Terminal*.

```bash
cd ~
mkdir chromium
cd chromium
fetch --nohooks chromium --nosvn=True
```


# Handling fetch Errors

The fetch command may fail if either your Internet connection or Google's
server goes down during the (very long) download. If that happens, remove
everything and try again.

```bash
cd ~
rm -rf chromium
```


# Get Libraries and Tools

On OS X, XCode contains all the tools you need to build Chromium.

On Ubuntu, run the commands below in *Terminal*.

```bash
cd ~/chromium/src
./build/install-build-deps.sh
```

For other Linux distributions, find the relevant section under
*Distribution-specific Notes* in
[Chromium's Linux Build Prerequisites wiki page](https://code.google.com/p/chromium/wiki/LinuxBuildInstructionsPrerequisites),
and follow the instructions there. The official instructions tend to be out of
date, so read through the comments section on the page for updates.


# Build Chromium

The Chromium build process takes up a few hours and all the CPU cycles that
your machine has to spare, causing its fans to blow at full speed. Make sure
this is appropriate in your environment.

Run the commands below in *Terminal* to build Chromium.

```bash
cd ~/chromium
gclient runhooks --force
cd src
ninja -C out/Debug chrome
```

## Dealing with Build Errors

If you're unlucky, you might get build errors. This usually happens when some
tool or library (such as the compiler installed by Xcode) is updated, and
Chromium hasn't caught up with the changes. Before trying anything else, update
the source code using the commands below, then run the build commands above
again.

```bash
cd ~/chromium
git pull && gclient sync
```

If you're particularly unlucky, you'll have to figure out the error by
yourself. It usually comes down to missing a library or tool.

On OSX, setting up [homebrew](http://brew.sh/) is the easiest way to get
missing libraries. On Ubuntu,
[auto-apt](https://help.ubuntu.com/community/AutoApt) will help you figure out
which package you need to install to get a missing file.


# Set Up the Sandbox

Chromium uses sandboxing to reduce the amount of damage that a security
vulnerability can cause. You don't need to understand how it works right now,
but you do need to run the commands below once to get your Chromium build
running.

```bash
cd ~/chromium/src
ninja -C out/Debug chrome_sandbox
sudo chown root:root out/Debug/chrome_sandbox
sudo chmod 4755 out/Debug/chrome_sandbox
```


# Run Chromium

If you made it this far, your development machine is all set up for hacking on
Chromium. Check your work by running the Chromium binary that you have just
built.

On OS X, run the commands below.

```bash
cd ~/chromium/src
out/Debug/Chromium.app/Contents/MacOS/Chromium
```

On Linux, use these commands instead.

```bash
cd ~/chromium/src
out/Debug/chrome
```

Congratulations! You are now ready to change the world!


# References

I used information from the Chromium guides below.

* [Using depot_tools](http://www.chromium.org/developers/how-tos/depottools)
* [Get the Code](http://www.chromium.org/developers/how-tos/get-the-code)
* [Using Git](https://code.google.com/p/chromium/wiki/UsingGit)
* [Linux SUID Sandbox Development](https://code.google.com/p/chromium/wiki/LinuxSUIDSandboxDevelopment)
* [WebKit/Blink Conversion Cheatsheet](http://www.chromium.org/blink/conversion-cheatsheet)
* [Mac Build Instructions](https://code.google.com/p/chromium/wiki/MacBuildInstructions)
* [Linux Build Instructions](https://code.google.com/p/chromium/wiki/LinuxBuildInstructions)

