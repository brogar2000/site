# The Yggdrasil Screen Reader Project

## What is Yggdrasil?

Accessibility on Linux has historically been under-developed, under-maintained, and, therefore, gained a
reputation of being quite painful to use as a daily driver among disabled people. We want to change that.

Yggdrasil is a new project that aims to create a better Linux screen reader, written in [Rust][rust]. Through
this project, we aim to provide a better screen reading experience than the one we currently have in [Orca][orca].
A screen reader with all the modern features a Windows or macOS user would expect from their computers, some of which are outlined below.

[orca]: https://wiki.gnome.org/Projects/Orca
[rust]: https://rust-lang.org/

## Features

### First-class addon support

We want our users to be able to do anything they wish with our screen reader. Improving support for specific
programs should ideally be doable without needing to modify yggdrasil's core code.

We plan to make it easy to write addons primarily in [Lua][lua], with the option to use other languages such as
[Rust][rust] in the future for higher performance.

[lua]: https://www.lua.org/

### Flexible input and gesture system

Likewise, adding new functionality such as new keyboard commands should be easy enough, without needing to
tinker with the source code of the screen reader. Manual editing of the configuration files as well as an
intuitive grafical program to modify gestures and define new actions will be available.

### Modern navigation patterns

In a perfect world, apps would just be accessible automatically, However, that's almost never the case. Therefore, visually impaired people rely on additional tools provided by
their screen reader, such as object navigation, a way to explore the accessibility tree of an app without relying
on focus, being then able to discover unfocusable elements that have important information nevertheless. We
have this in [NVDA][nvda] and other screen readers, but not in Orca and as such, not in linux. Yggdrasil will
address that.

[nvda]: https://www.nvaccess.org/

## Why the Name?

In norse mythology, [yggdrasil][wikipedia] is a huge tree around which the nine worlds gravitate. It's seen as a
unifyer of everything, since its roots reach to the center of the earth. It's so large that parts of it exist in
other universes.

Similarly to the tree of life that unifies everything, We believe accessibility should unify people from across the globe to enjoy a product, with disability no longer
being an impediment to the person efficiently using it. They should be able to use technology as well as, if not
better than a non-disabled individual.

[wikipedia]: https://en.wikipedia.org/wiki/Yggdrasil

## announcements and news

Important announcements and news about Yggdrasil's development will be published here.

### Minimal speech-dispatcher Bindings Completed

As of afew days ago, a [minimal binding to speech-dispatcher][spd-rs] (the TTS system for Linux) was created, which
wraps the C functions provided by [Nolan's `speech-dispatcher-sys` crate][spd-sys] in a safe Rust API. That means the
speech component of Yggdrasil can now be implemented after the at-spi components are completed, as well as in other Rust
projects that need Linux only TTS, or want more control than the [tts crate][tts-rs] gives them.

These bindings are far from complete, but do provide the essential functionality, such as speaking text, characters and
keys, and getting and setting voice parameters such as rate, pitch, and volume.

In our view, that can't mean anything else but one step closer to the first Yggdrasil prototype!

[spd-rs]: https://github.com/yggdrasil-sr/tts_subsystem
[spd-sys]: https://github.com/ndarilek/speech-dispatcher-sys

    [tts-rs]: https://github.com/ndarilek/tts-rs

## Under Active Development

This website as well as the software itself is under active development, please check back at a later stage to
see more exciting news and documentation!

## Website / Documentation License

Copyright (c) 2021 the Yggdrasil contributors.

Permission is granted to copy, distribute and/or modify this document
under the terms of the GNU Free Documentation License, Version 1.3
or any later version published by the Free Software Foundation;
with no Invariant Sections, no Front-Cover Texts, and no Back-Cover Texts.
A copy of the license is included in the section entitled ["GNU
Free Documentation License"](fdl-1.3.md).
