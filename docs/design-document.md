# Design of the Yggdrasil Screen Reader

## Event Based Design

Since [node.js](https://nodejs.org/en/) was introduced to the market, the web space has enjoyed more scalable and responsive systems, due to Node's event driven architecture, where every system is loosly coupled to one another, where asyncronous programming, events and messages govern the program flow, therefore making traditional performance bottlenecks like disk i/o, database access and others nearly insignificant, since access to such resources is properly managed and interested parties are notified whenever such a resource is free to use.  
Inspired by this design, we decided to make the screen reader as event based as possible. Eventually the goal is to be able to abstract away the event sources in interfaces implementable by other components, therefore paving the way for alternative input methods.
For this to be doable, we'd need roughly the following to be implemented:

An event manager(runs on the main thread)

: this is the central hub for events. From here, events are received by system services like at-spi, atk and so on, then propagated to the components interested in that particular event. Note: Each component should register with the hub for every event it wants to recieve.  
The event hub should also reference the at-spi registry, and should register all the events the entire screen reader needs, then propagate them to the appropriate components. Running on another thread should not only be discouraged, but actively prohibited, for example by not implementing send and sync.

Component protocol(can run on multiple threads)

: this is the protocol every component should adhere to. It should include methods for sending and receiving events. Basically, every component should be thread-safe, the only way they communicate to the event hub being through a channel, for example rust's mpsc. The hub holds all the events in an array of data structures that, among other things, include the sending half of an mpsc channel for each of the registered components.

Keyboard/alternative input method component, UI component, speech component, etc

: specific implementations of the component protocol. Each component should deal with one, and only one, subsystem of the screen reader, sending proper events to the event hub for further propagation and handling by other systems if needed, for example the keyboard handler sends insert+q>goes to hub>goes to gesture manager>goes to quit>resource cleanup and quit program.

## Speech

As we want to make sure our screen reader integrates with the linux ecosystem and the existing tools as much as possible, we decided we will use speech dispatcher as the speech platform

Each speech related setting will be saved on a per screen reader, per user basis by default. However, we consider the possibility to include a utility that allows modification of the global speech dispatcher configuration. Whether this is achievable depends on how stable the speech-dispatcher config format is, and if we can reverse engineer it or find some kind of formal spec. We can use the [nom crate][nom] to parse it.

[nom]: https://crates.io/crates/nom

Though the default speech system will be speech dispatcher, we will try to make it possible to allow addons to define their own speech systems, though this is certainly lower priority, as writing a speech-dispatcher module is very easy and allows the synth to be used with other Linux tools.

## Object Navigation

Like in NVDA for Windows, this should follow the accessibility hierarchy as much as possible.
This should be done in the way users of screen readers with such features expect, a left/right command to navigate to the previous/next sibling of the object in focus, up to go to the parent object(if any), down to go to the children of an object(if any).

## Addons

**Note:** This section needs improvement and more discussion, however these are the ideas and plans so far

This section of the design document has concepts from the android app format(.apk.), as I think it's briliantly designed

an addon, before installation, is a zip archive that's digitally signed to verify the author's authenticity. It is composed of the following:

A manifest file, named always manifest.json

: Defines metadata about the addon, such as its name, version, author, minimal Ygdrassil version, capabilities, programming language, documentation links or local documentation, etc.

The addon bundle

: The code of the addon itself.  
depending on the language declared in the manifest, the addon will be searched in the following locations within the archive

    * for lua addons, it will be searched in lib/$addon_name.lua. The addon will be loaded through the lua module mechanism, a.k.a require.
    * for C or rust addons, it will be searched in lib/$target_tripple/lib$addon_name.so. This will be loaded through the dynamic linkage procedures in linux.

**Note:** We may allow addons to be written in other languages, such as C, using a custom addon subsystem that could be installed as an addon itself. However, installing such addons will warn the user that they could cause Yggdrasil to become unstable due to things like memory leaks or segmentation faults, and that we are not responsible for these crashes, which should be reported to the addon developers. This arrises from the fact that addons run in the same process as Yggdrasil, so if one crashes, it could bring Yggdrasil down with it. Languages that are memory safe, such as Rust, or garbage collected languages like Lua, won't produce a warning, as the risk of them crashing the entire screen reader are much lower, if not zero.

### Addon Distribution

We intend to provide a central distribution platform for addons, for easy installation and discovery by the users.  
Like any extension store, features like searching, filtering and so on will be available, as well as automatic updating upon the users choice.  
Since this is a public addon platform, we will have to make sure our users don't get infected due to the addons distributed through it, therefore

- In order to be published on the store, an addon needs to be digitally signed with a certificate that uniquely identifies the entity or organisation responsible for its development and distribution.
- a static analysis of the archive would comense whenever a new version is uploaded. This could include scanning of the manifest file for dubious sets of intitlements, a submition of the binary files to virustotal, etc.
- Of course, we will encourage developers to make use of unit testing, and down the road could provide a unit testing framework that simulates the Yggdrasil addon API that can be used to programatically test addon functionality.

However, with a specific setting in the screen reader, the user can allow the installation of extensions not distributed through the store. Then, the addon manager would allow browsing to an addon file and install it.

### Included Addons

In the spirit of modularity and collaboration potential, and because we want our addon api to be as bulletprufe, efficient to use and clearly documented as possible, most of the not strictly needed but useful components as comodity features(what is strictly not necesary to operate the core) will be included with the distribution as addons. Per the users choices, some of those can be disabled or even uninstalled, cutting down on the installed size.

## ocr

The ocr system will have two modes:

### NVDA Style OCR

The ocr results open in a different window, the user being able to click pieces of text for the mouse to perform a click at the desired location in the real window. This would also be the fastest way to perform even arbitrary ocr, as the ocr info is available to addons, so the window doesn't necesarily need to pop open, exactly like nvda.  
**this is under discussion, if it's indeed possible to do such things with the current open source accessibility stack we have.**

### Innovative OCR

This mode doesn't attempt to open a new window, instead it overlays the ocr within the current one.  
Based on the control's shape, position and color, yggdrasil will try to make an accessible tree for that window, recognising the controll type and using the captured text as the controll text if applicable. Of course, because of the error probability being so high, there would be a possibility for the user to define controlls manually based on the mouse position and providing the text, or make improvements to the already generated ocr result, then saving it as a file that can be distributed. Of course such tweeks will be resolution dependent, but probably the utility will try to correct it as much as possible to fit the application by finding the greattest similarity to both the text and coords and controll type from the provided database in the ocred text.

## Sound Themes (Earcons)

For a long time, mobile device users have had earcons, a sound representation of each control and element type on the current interface. Due to that, such users don't need to wait for the speech to arrive to the controll type, as it's obvious what it is based on the specific sound.
NVDA uses an addon called [audio themes][audio-themes-nvda-addon] together with unspoken to deliver this functionality which, while not perfect and can lag sometimes, at least tryes to accomplish the same level of efficiency as the mobile experience and manages for the most part.  
While proliminary support for audio has been implemented in Orca, it hasn't been expanded on since, and as far as we know, the only thing that uses it is progress bar beeps.
Yggdrasil will implement this as an addon that's part of the distribution, since its another unnecesary feature for the functioning of the screen reader, even though it increases productivity.
Eventually, it would also be nice to support positional audio for earcons, so the user knows where the control they're interacting is on screen.

[audio-themes-nvda-addon]: https://addons.nvda-project.org/addons/AudioThemes.en.html

## Gesture System

The system needs to be as flexible as possible, so the gesture system will be an event component implementing the above mentioned protocol.  
First, it takes all the raw user actions as events from the hub, registering them all, like key presses, touch gestures and other user-generated commands through alternative input methods.
Then, it reads the gestures configuration file, or maybe database.
When it finds the gesture corresponding to the event it got from the hub, it reads the action for that, constructs the appropriate event, sending it to the event hub to be processed and dispatched to the correct component.

## Links to Currently Available Documentation

- [at-spi API documentation](https://www.manpagez.com/html/libatspi/libatspi-2.10.1/ch01.php)
- [at-spi general overview](https://www.freedesktop.org/wiki/Accessibility/AT-SPI2/), with several links.
- [at-spi examples](https://github.com/infapi00/at-spi2-examples), including Javascript, C and Python.
- [Examples using exclusively py-atspi2](https://www.freedesktop.org/wiki/Accessibility/PyAtSpi2Example/), the human-generated bindings, not the ones generated via object introspection.
- [Rust bindings for at-spi](https://github.com/mcb2003/atspi-rs)
- [Rust bindings to speech dispatcher, speech-dispatcher-sys](https://crates.io/crates/speech-dispatcher-sys/0.5.2). Bindings autogenerated with rust-bindgen, written by nolan
- [speech-dispatcher library documentation](http://htmlpreview.github.io/?https://github.com/brailcom/speechd/blob/master/doc/speech-dispatcher.html).
- [The Speech Synthesis Interface Protocol (SSIP) API documentation](http://htmlpreview.github.io/?https://github.com/brailcom/speechd/blob/master/doc/ssip.html).
- [Liblouis bindings for rust](https://github.com/whentze/liblouis-rust), for braille tables and multiple braille codes, including those for the majority of languages
- [Libbrlapi reference manual](https://brltty.app/doc/Manual-BrlAPI/English/BrlAPI.html).
