
# renderview spec

A protocol for interactive rendering surfaces.

*Last update: 09-05-2025*

*This spec was previously known as the jupyter_rfb event spec, but was rolled into a separate project.*



## Introduction

RenderView defines a simple specification for a renderable surface that emits
structured input events (pointer, keyboard, resize) and displays frames. It can
be implemented in most programming languages, browsers, native applications,
remote renderers, notebooks, etc.

Relevant Links:

* Repo: [https://github.com/pygfx/renderview](https://github.com/pygfx/renderview)
* Reference implementation and basis for browser-based implementations: <br> [renderview.js](renderview.js) and [renderview.css](renderview.css)
* The [tester.html](tester.html)

This spec if used by:

* [rendercanvas](https://github.com/pygfx/rendercanvas)
* [jupyter_rfb](https://github.com/vispy/jupyter_rfb)



## Controlling the view

The way that the view (a.k.a. canvas, surface, widget) can be controlled is left up to
the implementation. This spec does not say much about it, except for the
following suggestions for settable things:

* The size in logical pixels.
* The cursor.
* The title.

In a browser context:

* The size as css strings.
* Whether the view is resizable.
* Whether the title is shown.



## Event objects

Event objects are dictionaries ('object' in JS, 'dict' in Python) with a few pre-defined keys.
Implementations can wrap these into types objects to improve usability. But since the dictionaries
are well-defined, they can always be serialized to json, and send to other programming environments.



## Event types


### resize

This event is emitted when the canvas changes size.


Fields:

* `event_type`: 'resize'
* `width`: The width in logical pixels.
* `height`: The height in logical pixels.
* `pwidth`: The width in physical pixels.
* `pheight`: The height in physical pixels.
* `pixel_ratio`: The pixel ratio between logical and physical pixels.
* `time_stamp`: A timestamp in seconds.


### close

This event is emitted when the canvas is closed (i.e. destroyed).

Fields:

* `event_type`: 'close'
* `time_stamp`: A timestamp in seconds.


### show

This event is emitted when the canvas is shown. The implementation may
optionally emit an initial 'show' event; the canvas is initially assumed to be
shown. If this is not the case, the implementation should emit a 'hide' event.

Fields:

* `event_type`: 'show'
* `time_stamp`: A timestamp in seconds.


### hide

This event is emitted when the canvas is hidden. It can e.g. be minimized or scrolled out of view.

Fields:

* `event_type`: 'hide'
* `time_stamp`: A timestamp in seconds.


### pointer_down

This event is emitted when the user interacts with mouse, touch or other pointer devices, by pressing it down.

Fields:

* `event_type`: 'pointer_down'
* `x`: The horizontal position of the pointer within the canvas.
* `y`: The vertical position of the pointer within the canvas.
* `button`: The button to which this event applies.
* `buttons`: A tuple of buttons being pressed down.
* `modifiers`: A tuple of modifier keys being pressed down.
* `ntouches`: Experimental.
* `touches`: Experimental.
* `time_stamp`: A timestamp in seconds.

The mouse buttons are defined as follows, which is different from JavaScript (although close to the JS `buttons` bitmask):

* 0: No button.
* 1: Left button.
* 2: Right button.
* 3: Middle button
* 4 - 9: etc.

See the section on [event capturing](#event-capturing) for detailed behavior.


### pointer_up

This event is emitted when the user releases a pointer.

Fields:

* `event_type`: 'pointer_up'
* ... the other fields are the same as the pointer_down event.


### pointer_move

This event is emitted when the user moves a pointer.

In remote contexts it is recommended to throttle this event to avoid spamming the IO.

Fields:

* `event_type`: 'pointer_move'
* ... the other fields are the same as the pointer_down event.


### pointer_enter

This event is emitted when the user moves a pointer into the boundary of the canvas.

Fields:

* `event_type`: 'pointer_enter'
* `time_stamp`: A timestamp in seconds.


### pointer_leave

This event is emitted when the user moves a pointer out of the boundary of the canvas.

Fields:

* `event_type`: 'pointer_enter'
* `time_stamp`: A timestamp in seconds.


### double_click

This event is emitted on a double-click.

Fields:

* `event_type`: 'pointer_down'
* `x`: The horizontal position of the pointer within the canvas.
* `y`: The vertical position of the pointer within the canvas.
* `button`: The button to which this event applies.
* `buttons`: A tuple of buttons being pressed down.
* `modifiers`: A tuple of modifier keys being pressed down. See section below for details.
* `time_stamp`: A timestamp in seconds.


### wheel

This event is emitted when the mouse-wheel is used (scrolling), or when scrolling / pinching on the touchpad / touchscreen.

In remote contexts it is recommended to throttle this event to avoid spamming the IO.

Fields:

* `event_type`: 'wheel'
* `dx`: The horizontal scroll delta (positive means scroll right).
* `dy`: The vertical scroll delta (positive means scroll down or zoom out).
* `x`: The mouse horizontal position during the scroll.
* `y`: The mouse vertical position during the scroll.
* `buttons`: aA tuple of buttons being pressed down.
* `modifiers` *: A tuple of modifier keys being pressed down.
* `time_stamp`: A timestamp in seconds.

Similar to the JS wheel event, the values of the deltas depend on the
platform and whether the mouse-wheel, trackpad or a touch-gesture is
used. Also, scrolling can be linear or have inertia. As a rule of
thumb, one "wheel action" results in a cumulative ``dy`` of around
100. Positive values of ``dy`` are associated with scrolling down and
zooming out. Positive values of ``dx`` are associated with scrolling
to the right. (A note for Qt users: the sign of the deltas is (usually)
reversed compared to the QWheelEvent.)

On MacOS, using the mouse-wheel while holding shift results in horizontal
scrolling. In applications where the scroll dimension does not matter,
it is therefore recommended to use `delta = event['dy'] or event['dx']`.


### key_down

This event is emitted when a key is pressed down.

Fields:

* `event_type`: 'key_down'
* `key`: The key being pressed as a string.
* `modifiers`: A tuple of modifier keys being pressed down.
* `time_stamp`: A timestamp in seconds.

The key names follow the [browser spec](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent).

* Keys that represent a character are simply denoted as such. For these the case matters: "a", "A", "z", "Z" "3", "7", "&", " " (space), etc.
* The modifier keys are: "Shift", "Control", "Alt", "Meta".
* Some example keys that do not represent a character: "ArrowDown", "ArrowUp", "ArrowLeft", "ArrowRight", "F1", "Backspace", etc.
* When a key is held down, the events should *not* repeat.


### key_up

This event is emitted when a key is released.

* `event_type`: 'key_up'
* `key`: The key being released as a string.
* `modifiers`: A tuple of modifier keys being pressed down.
* `time_stamp`: A timestamp in seconds.


### char

This event is emitted when a character is typed.
An experimental event to support text-editing. The spec for this event will likely change in the future.

Fields:

* `data`: The Unicode character being typed.
* `input_type`: The typing action, e.g. 'insertText', 'insertCompositionText', 'deleteBackwards'.
* `is_composing`: Whether the inserted text is being composited (i.e. temporary).
* `repeat`: Whether this is a repeated event from a key being held down.



## Details


### Time stamps

The reference for the used time stamps is not defined by this spec. They could be Unix timestamps or something that starts at application start.
In multi-process environments you can therefore not compare the timestamps to e.g. `time.time()`.


### Coordinate frame

The coordinate frame is defined with the origin in the top-left corner.
Positive `x` moves to the right, positive `y` moves down.


### Event capturing

The `pointer_move` event only occurs when the pointer is over the canvas,
unless a button is down (i.e. dragging). The `pointer_down` event can only
occur inside the canvas, the `pointer_up` can occur outside of the canvas.

Some events only work when the canvas has focus within the application
(i.e. having received a pointer down).
This applies to the `key_down`, `key_up`, and `wheel` events.


### Application focus

* When the application does not have focus, it does not emit any pointer events.
* When the application loses focus, a `pointer_leave` event is emitted, and
also a `pointer_up` event if a button was held down.
* When the application regains focus, an enter event is emitted if the pointer
if over the canvas. This may not happen until the pointer is moved.
* If the application regained focus by clicking on the canvas, that click does
not result in pointer events (down, move, nor up).


### Event throttling

To avoid straining the IO, certain events can be throttled. Their effect
is accumulated if this makes sense(e.g. wheel event).



<br><br>

*This file is dedicated to the public domain under CC0 1.0.*
