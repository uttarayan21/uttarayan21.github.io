---
layout: post
title: My current setup as of 4'th May of 2020
author: uttarayan21
---


## Terminal

I use [alacritty](https://github.com/alacritty/alacritty) as my terminal.
It is a superfast terminal emulator written in rust and is gpu accelerated.
It however doesnt support any sort of graphics inside the terminal.

My secondary terminal is st or suckless terminal my fork of which contains vim like keys for navigation in the scrollback
buffer and also alt+c and alt+v mapped to copy and paste. It also has the boxdraw, copyurl, alpha patches and a custom color patch
to match the default color of termite, patches applied to it.

I also use a few replacements for the common gnu tools :

* rg or [ripgrep](https://github.com/sharkdp/ripgrep) instead of grep
* [fd](https://github.com/sharkdp/fd) instead of find
* [exa](https://github.com/ogham/exa) instead of ls
I use [zsh-abbr](https://github.com/olets/zsh-abbr) to alias them to their respective gnu tools.


I use neomutt as my email client and mbsync to download them locally and notmuch to search through mail

I use tmux for terminal multipling. It is really helpful to just close the terminal and reconnect to it outside home from an ssh session.

You can find my dot files on here
## Editor

Currently my main editor is [neovim](https://www.github.com/neovim/neovim). It
has learning curve but it can really speedup writing and editing code
I also have mapped my capslock to control so it gives me most of the fuctionality
of vim in my home row and not really move my hands too much
## Shell

As my shell I use zsh with [oh-my-zsh](https://github.com/ohmyzsh/ohmyzsh)  zsh-autosuggestion zsh-syntaxhighlighting and [zsh-abbr](https://github.com/olets/zsh-abbr).

## Distro

As for my distro I primarily use ArchLinux. I have been using arch for about 4 years now and I have been really satisfied with it.
The best thing about arch is that it forced me to learn linux and without arch I would still have not learned linux like I do now.
Also one of the primary reasons for my love of arch is the [aur](https://aur.archlinux.org/). I personally find the aur probably the best features of archlinux.

## WM

I usually prefer tiling window managers
I am curently using bspwm as my window manager. It is lightweight but comes with minimal defaults
so it is a bit of work to configure but is worth it in the end. I use picom (formerly compton) as my compositor.

There is a small program called unclutter to hide the mouse when I'm typing and don't want my touchpad to interfere

## DM

For my dm I use ly, it is a super lightweight ncurses based dm, and can launch xorg as well as wayland based Window Managers
