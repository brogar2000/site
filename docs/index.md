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

###the first prototype has been released!

We know it has been a long time, however we are delited to inform you that the first yggdrasil prototype is up on github, with a very early alpha stage build for anyone curious enough to try it out.  
the screen reader can't do much at the moment, however this is what it can do so far

* like any normal screen reader, it can read most components exposed by the linux accessibility interface, such as buttons, checkboxes, radio buttons, etc.  
an exception to this is the webview controll and interacting with text boxes, though the latter might be implemented around the end of the week, if everything goes well.

* some web elements can be navigated through and interacted with, for example links, buttons, checkboxes, forms, etc, even though text inside the webview and quick navigation doesn't work yet.

As we all know, this is the first release of an alpha quality software, so feel free to report bugs by opening issues against the repository. However, here are some known issues that we will attempt to fix in the next couple releases, so no need to report those

* sometimes, the name or text of a controll might not be announced, the speech saying something like ":button", mark the punctuation.  
This is most likely caused by at-spi sending events to yggdrasil before the controlls actually finished loading, so the controll names and other attributes might not be filled in by the time yggdrasil gets the event, though they aren't null since the data exists somewhere, just didn't exist then.

* if a controll uses something else than name or text to label itself, for example tooltips, yggdrasil won't pick those as valid ways of labeling, so it will either speak the same as described above, or it could even say unlabeled in some cases

* currently, there's no other way to quit the screen reader but by sending it a keyboard interrupt sygnal, or killing the process in another way. This is so because of the binding generators we work with, they make stuff more complicated, so no keyboard handlers in this release.

* on some machines, it's reported that the screen reader doesn't read window titles when cycling between them with alt+tab. Currently, we don't know why this is, but we're investigating still.

The road to here was a very rough one indeed, however we are excited we could bring this to you, as demo quality as this is. This means much to us, it proves progress is being done, against all odds.  
If you want to see more content from us, consider visiting our website more often, this way you won't lose anything. Furthermore, you will get information from the source of the project, so you can confirm its authenticity for yourself

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
