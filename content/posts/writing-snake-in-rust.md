---
layout: post
title: "Writing Snake in rust"
<!-- date: Mon, 01 Feb 2021 04:45:42 +0530 -->
categories: rust snake
published: false
---

## Snake in rust

I'm learning rust so I wrote a [snake][snake-rs] game on rust using the crate [ncurses][ncurses-rs]
It has vim like keybindings so it can be used to train muscle memory for new vim users.

This was my first program using ncurses.
The snake's body co-ordinates are stored in a `std::collections::LinkedList`
On every tick the head is drawn and the tail is removed (overwritten with `0x20` or space)

The main issue I faced while writing it was how to seperate the backend of the game from the frontend of the game ([ncurses][ncurses]).
I want to do a rewrite so that the crate is a library and can be used with any frontend.

[snake-rs]: https://github.com/uttarayan21/snake
[ncurses-rs]: https://docs.rs/ncurses
[ncurses]: https://invisible-island.net/ncurses
