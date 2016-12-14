---
layout: post
title: Developer Tools - Jack
excerpt_separator: <!-- /excerpt -->
description: Developer tools used by Mexico Insurance Online's lead developer.
---

We developers **love** our tools; here are Jack's favorites.
<!-- /excerpt -->
Tools can make all the difference in the world to a developer.  Everything from
the keyboard you physically type on, to the browser you test code in.  It all
lends a hand in making developers as efficient as possible.

## Physical Interface

Let's get it out of the way... I develop on a 15" MacBook Pro.  Why, you may
ask?  I find that it is much more efficient to develop on a machine as close
to the server as possible and MacOS is a close cousin to the Linux based
servers where MIO runs.  MacOS also alleviates the hassle of configuring
monitors, networks, keyboards, mice, etc, that you may find in a Linux
environment.

I also use gaming accessories for things like a
[Razer](http://www.razerzone.com/) Blackwidow Ultimate keyboard, and an old
Razer Naga mouse I had lying around.  I find they are very comfortable to me, 
and a joy to use.

## Editor

In the last year I've switched from [vim](http://www.vim.org/) to Github's
[Atom](https://atom.io/) editor.  Atom provides the same great keyboard-only
experience as vim, but with community plugins and some additional features.

## Terminal

In order to start servers and really get around in a Unix-like OS (such as
MacOS), a good terminal is required.  MacOS's built in Terminal is fantastic,
but I find [Hyper](https://hyper.is/)'s customization to be worth the switch.

I also run ZSH for my shell, and
[Oh-My-ZSH](https://github.com/robbyrussell/oh-my-zsh) to add tools to the
command line.

## Virtualization

MIO uses a lot of tools in many different languages. In order to keep these all
neat and tidy, we make extensive use of [Docker](https://www.docker.com/). 
Docker allows me to have different development environments running on the
same machine without clomping all over each other.  As an example, this blog
is written using a version of Ruby that would conflict with the version we use
to develop custom UI's for some agents.  Before Docker, it could take hours of
work to switch between those two projects.  With Docker I can ignore that
dependancy and just start working.

