# design of the yggdrasil screen reader

## event based design

<<<<<<< HEAD
Since [node.js](https://nodejs.org/en/) was introduced to the market, the web server space enjoyed more scalable and responsive systems, that's because of its endorsing of an event driven development, where every system is loosly coupled to one another, where asyncronous programming, events and messages govern the program flow, therefore making traditional performance bottlenecks like disk i/o, database access and others nearly insignificant, since access to such resources is properly managed and interested parties are notified whenever such a resource is free to use.  
=======

Since [node.js](https://nodejs.org/en/) was introduced to the market, the internet has enjoyed more scalable and responsive systems, due to Node's event driven architecture, where every system is loosly coupled to one another, where asyncronous programming, events and messages govern the program flow, therefore making traditional performance bottlenecks like disk i/o, database access and others nearly insignificant, since access to such resources is properly managed and interested parties are notified whenever such a resource is free to use.  
>>>>>>> 8d654f6dc4d033a1caccafd0ff3075452a6b74a8
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

As we want to make sure our screen reader integrates with the linux ecosystem and the existing tools as much as possible, we decided we will use speech dispatcher as the speech platform  
<<<<<<< HEAD
Each speech related setting will be saved on a per screen reader, per user basis by default. However, we consider the possibility to include a utility that allows modification of the global speech dispatcher configuration. Note however, such a program would require root privileges for full functionality, otherwise it would only modify the user-specific speech dispatcher configuration.  
Also, though the default speech system would be speech dispatcher, we will try to make it possible to allow addons to define their own speech systems, though this would require special intitlements in the manifest.
=======

Each speech related setting will be saved on a per screen reader basis, per user basis by default. However, we will include a utility to allow modification of the speech dispatcher configuration files.
Also, though the default speech system would be speech dispatcher, we will try to make it possible to allow addons to define their own speech systems. This, however, will be a much lower priority, as it is very easy to write a speech-dispatcher module.
>>>>>>> 8d654f6dc4d033a1caccafd0ff3075452a6b74a8

## object navigation

object navigation, like in NVDA for Windows, should follow the accessibility hierarchy as much as possible. This should be done in the way users of screen readers with such features expect, a left/right command to navigate to the previous/next ansester of the object in focus, up to go to the parent object(if any), down to go to the children of an object(if any).

## addons

<<<<<<< HEAD
Note. This section needs improvement and more discussion, however these are the common guidelines so far  
Disclaimer, This section of the design document has concepts from the android app format(.apk.), as I think it's briliantly designed  
an addon, before installation, is a zip archive that's digitally signed to verify the author's authenticity. Also, it's composed of the following:  
a manifest file, named always manifest.json
=======

**Note:** This section needs improvement and more discussion, however these are the common guidelines so far  
This section of the design document has concepts from the android app format(.apk.), as I think it's briliantly designed  
an addon, before installation, is a zip archive that's digitally signed to verify the author's authenticity. Also, it's composed of the following:

A manifest file, named always manifest.json
>>>>>>> 8d654f6dc4d033a1caccafd0ff3075452a6b74a8

: This defines the name of the addon, version, author, minimal ygdrassil version, capabilities, programming language(when applicable), documentation links or local documentation, etc.

The addon bundle

: this is the code of the addon itself.  
depending on the language declared in the manifest, the addon will be searched in the following locations within the archive:

    - for lua addons, it will be searched in lib/$addon_name.lua. The addon will be loaded through the lua module mechanism, a.k.a require.
    - for C or rust addons, it will be searched in lib/$target_tripple/lib$addon_name.so. This will be loaded through the dynamic linkage procedures in linux.

## OCR

<<<<<<< HEAD
note. When the manifest file specifies a language that's not one of the official languages, for example the C language or another language that calls C bindings, the user will be warned that installing of this extension can cause harm to the system, since it's written in an unsafe/unsupported language.

### distributions of add-ons

We intend to provide a central distribution platform for addons, for easy installation and discovery by the users.  
Like any extension store, features like searching, filtering and so on will be available, as well as automatic updating upon the users choice.  
Since this is a public addon platform, we will have to make sure our users don't get infected due to the addons distributed through it, therefore

* In order to be published on the store, an addon needs to be digitally signed with a certificate that uniquely identifies the entity or organisation responsible for its development and distribution.
* a static analysis of the archive would comense whenever a new version is uploaded. This could probably include scanning of the manifest file for dubious sets of intitlements, a submition of the binary files to virustotal, etc.

However, with a specific setting in the screen reader, the user can allow the installation of extensions not distributed through the store. Then, the addon manager would allow browsing to an addon file and install it.  

### included addons

In the spirit of modularity and collaboration potential, as well as because  we want our addon api to be as bulletprufe, efficient to use and clearly documented as possible, most of the not strictly needed but useful components as comodity features(what is strictly not necesary to operate the core) will be included with the distribution as addons. Per the users choices, some of those can be disabled or even uninstalled, cutting down on the installed size considerably.

## ocr

The ocr system will have two modes:  

### nvda stile ocr

The ocr results open in a different window, the user being able to click pieces of text for the mouse to perform a click at the desired location in the real window. This would also be the fastest way to perform even arbitrary ocr, as the ocr info is available to addons, so the window doesn't necesarily need to pop open, exactly like nvda.  
**this is under discussions, if it's indeed possible to do such things with the current open source technologies we have**

### innovative ocr

This mode doesn't attempt to open a new window, instead it overlays the ocr within the current one.  
Based on the control's shape, position and probably color, yggdrasil will try to make an accessible tree for that window, recognising the controll type and using the captured text as the controll text if applicable. Of course, because of the error probability being so high, there would be a possibility for the user to define controlls manually based on the mouse position and providing the text, or make improvements to the already generated ocr result, then saving it as a file that can be distributed. Of course such tweeks will be resolution dependent, but probably the utility will try to correct it as much as possible to fit the application by finding the greattest similarity to both the text and coords and controll type from the provided database in the ocred text.

## sound themes, earcons

For a long time, mobile device users had earcons, a sound representation of each control and element type on the current interface. Due to that, such users don't need to wait for the speech to arrive to the controll type, as it's obvious what it is based on the specific sound it makes.  
NVDA uses an addon called sound themes together with unspoken to deliver this functionality which, while not perfect and can lag sometimes, at least tryes to accomplish the same level of efficiency as the mobile experience and manages for the most part.  
Orca however, doesn't have this support, yggdrasil will fix that deficiency as well.  
This would be implemented as an addon that's part of the distribution, since its another unnecesary feature for the functioning of the screen reader, just a comodity, even though it increases productivity.

## gesture system

=======
The OCR system will have two modes:

NVDA stile OCR

: The OCR results open in a different window. The user will be able to click pieces of text for the mouse to perform a click at the desired location in the real window. This would also be the fastest way to perform even arbitrary OCR, as the OCR info is available to addons, so the window doesn't necesarily need to pop open, exactly like nvda.

    **this is under discussions, if it's indeed possible to do such things with the current open source technologies we have**

Innovative OCR

: This mode doesn't attempt to open a new window, instead it overlays the OCR within the current one.  
Based on the control's shape, position and probably color, yggdrasil will try to make an accessible tree for that window, recognising the controll type and using the captured text as the controll text if applicable. Of course, because of the error probability being so high, there would be a possibility for the user to define controlls manually based on the mouse position and providing the text, or make improvements to the already generated OCR result, then saving it as a file that can be distributed. Of course such tweeks will be resolution dependent, but probably the utility will try to correct it as much as possible to fit the application by finding the greattest similarity to both the text and coords and controll type from the provided database in the OCRed text.

## Gesture System

>>>>>>> 8d654f6dc4d033a1caccafd0ff3075452a6b74a8

The system needs to be as flexible as possible, so the gesture system will be an event component implementing the above mentioned protocol.  
First, it takes all the raw user actions as events from the hub, registering them all, like key presses, touch gestures and other user-generated commands through alternative input methods. Then, it reads the gestures configuration file, or maybe database. When it finds the gesture corresponding to the event it got from the hub, it reads the action for that, constructs the appropriate event, sending it to the event hub to be processed and dispatched to the correct component.

## Links to Currently Available Documentation

* [at-spi API documentation](https://www.manpagez.com/html/libatspi/libatspi-2.10.1/ch01.php)
* [at-spi general overview](https://www.freedesktop.org/wiki/Accessibility/AT-SPI2/), with several links.
* [at-spi examples](https://github.com/infapi00/at-spi2-examples), including Javascript, C and Python.
* [Examples using exclusively py-atspi2](https://www.freedesktop.org/wiki/Accessibility/PyAtSpi2Example/), the human-generated bindings, not the ones generated via object introspection.
* [Rust bindings for at-spi](https://github.com/mcb2003/atspi-rs)
* [Rust bindings to speech dispatcher, speech-dispatcher-sys](https://crates.io/crates/speech-dispatcher-sys/0.5.2). Bindings autogenerated with rust-bindgen, written by nolan
* [speech-dispatcher library documentation](http://htmlpreview.github.io/?https://github.com/brailcom/speechd/blob/master/doc/speech-dispatcher.html).
* [The Speech Synthesis Interface Protocol (SSIP) API documentation](http://htmlpreview.github.io/?https://github.com/brailcom/speechd/blob/master/doc/ssip.html).
* [Liblouis bindings for rust](https://github.com/whentze/liblouis-rust), for braille tables and multiple braille codes, including those for the majority of languages
* [Libbrlapi reference manual](https://brltty.app/doc/Manual-BrlAPI/English/BrlAPI.html).
