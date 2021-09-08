# design of the yggdrasil screen reader

## event based design

Since [node.js](https://nodejs.org/en/) was introduced to the market, the internet has enjoyed more scalable and responsive systems, due to Node's event driven architecture, where every system is loosly coupled to one another, where asyncronous programming, events and messages govern the program flow, therefore making traditional performance bottlenecks like disk i/o, database access and others nearly insignificant, since access to such resources is properly managed and interested parties are notified whenever such a resource is free to use.  
Inspired by this design, we decided to make the screen reader as much event based as possible. Eventually the goal is to be able to abstract away the event sources in interfaces implementable by other components, therefore paving the way for alternative input methods.  
For this to be doable, we'd need roughly the following to be implemented:

an event manager(runs on the main thread)

: this is the central hub for events. From here, events are received by system services like at-spi, atk and so on, then propagated to the components interested in that particular event. Note: Each component should register with the hub for every event it wants to recieve.  
The event hub should also have a direct reference to the at-spi registry, as well as it should register for all the events the entire screen reader needs, then propagate them to the appropriate components. Running on another thread should not only be discouraged, but actively prohibited, for example by not implementing send and sync.

component protocol(can run on multiple threads)

: this is the protocol every component should adhere to. It should include methods for sending and receiving events. Basically, every component should be thread-safe, the only way they communicate to the event hub being through a channel, for example rust's mpsc. The hub holds all the events in an array of data structures that, among other things, include the sending half of an mpsc channel for each of the registered components.

keyboard/alternative input method component, UI component, speech component, etc

: specific implementations of the component protocol. Each component should deal with one, and only one, subsystem of the screen reader, sending proper events to the event hub for further propagation and handling by other systems if needed, for example the keyboard handler sends insert+q>goes to hub>goes to gesture manager>goes to quit>resource cleanup and quit program.

## speech

As we want to make sure our screen reader integrates with the linux ecosystem and the existing tools as much as possible, we decided we will use speech dispatcher as the speech platform.  
Each speech related setting will be saved on a per screen reader basis, per user basis by default. However, we will include a utility to allow modification of the speech dispatcher configuration files.
Also, though the default speech system would be speech dispatcher, we will try to make it possible to allow addons to define their own speech systems. This, however, will be a much lower priority, as it is very easy to write a speech-dispatcher module.

## object navigation

object navigation, like in NVDA for Windows, should follow the accessibility hierarchy as much as possible. This should be done in the way users of screen readers with such features expect, a left/right command to navigate to the previous/next ansester of the object in focus, up to go to the parent object(if any), down to go to the children of an object(if any).

## addons

**Note:** This section needs improvement and more discussion, however these are the common guidelines so far  
This section of the design document has concepts from the android app format(.apk.), as I think it's briliantly designed  
an addon, before installation, is a zip archive that's digitally signed to verify the author's authenticity. Also, it's composed of the following:

A manifest file, named always manifest.json

: This defines the name of the addon, version, author, minimal ygdrassil version, capabilities, programming language(when applicable), documentation links or local documentation, etc.

The addon bundle

: this is the code of the addon itself.  
depending on the language declared in the manifest, the addon will be searched in the following locations within the archive:

    - for lua addons, it will be searched in lib/$addon_name.lua. The addon will be loaded through the lua module mechanism, a.k.a require.
    - for C or rust addons, it will be searched in lib/$target_tripple/lib$addon_name.so. This will be loaded through the dynamic linkage procedures in linux.

## OCR

The OCR system will have two modes:

NVDA stile OCR

: The OCR results open in a different window. The user will be able to click pieces of text for the mouse to perform a click at the desired location in the real window. This would also be the fastest way to perform even arbitrary OCR, as the OCR info is available to addons, so the window doesn't necesarily need to pop open, exactly like nvda.

    **this is under discussions, if it's indeed possible to do such things with the current open source technologies we have**

Innovative OCR

: This mode doesn't attempt to open a new window, instead it overlays the OCR within the current one.  
Based on the control's shape, position and probably color, yggdrasil will try to make an accessible tree for that window, recognising the controll type and using the captured text as the controll text if applicable. Of course, because of the error probability being so high, there would be a possibility for the user to define controlls manually based on the mouse position and providing the text, or make improvements to the already generated OCR result, then saving it as a file that can be distributed. Of course such tweeks will be resolution dependent, but probably the utility will try to correct it as much as possible to fit the application by finding the greattest similarity to both the text and coords and controll type from the provided database in the OCRed text.

## Gesture System

The system needs to be as flexible as possible, so the gesture system will be an event component implementing the above mentioned protocol.  
First, it takes all the raw user actions as events from the hub, registering them all, like key presses, touch gestures and other user-generated commands through alternative input methods. Then, it reads the gestures configuration file, or maybe database. When it finds the gesture corresponding to the event it got from the hub, it reads the action for that, constructs the appropriate event, sending it to the event hub to be processed and dispatched to the correct component.

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
