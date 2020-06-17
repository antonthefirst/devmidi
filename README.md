# devmidi
A simple MIDI input wrapper that dovetails nicely into [Dear ImGui](https://github.com/ocornut/imgui).
It currently supports the two controllers I've found most useful in my own personal work:
* [Midi Fighter Twister](https://www.midifighter.com/#Twister) for knobs
* [Midi Fighter 3D](https://www.midifighter.com/#3D) for buttons

It's easy to hack it to add your own.

The lib currently lets you do the following:
* Bind float values to knobs and ImGui sliders simultaneously.
* Click the knobs for various convenient behavior like resetting to a default, printing, toggling.
* Bind buttons to ImGui buttons, checkboxes, and radio buttons.
* Read the raw data of the devices in a simple way.

## Motivation
I've found it very handy to use MIDI controllers for various tuning and debugging tasks during development:
* Color Grading - keep your eyes on the screen while tuning multiple parameters, without looking over at sliders or hunting with a mouse.
* Visualize Graphics Buffers - press and hold a button to temporarily visualize an internal buffer or texture (like depth, normals or other such info).
* Tune Gameplay Parameters - change common 'high frequency' tunings like speed, jump height, strengths.
* Toggle Between Versions - use buttons as hardware 'radio buttons' to quickly toggle between various versions of code to compare them.
* VR - do all of the above while wearing a VR headset, because hardware knobs and buttons can be operated easily without seeing them.

Bringing in a full blown MIDI lib into projects that I work on often feels like overkill (especially if it's a tiny personal one), so this is my attempt to remedy this and make it easy to just drop the functionality in. I've also found that simply binding to a MIDI controller has some downsides:
* You can't see the numeric value without logging it somewhere, and that adds some friction.
* You can't set an exact numeric value, which is useful sometimes.
* If your controller isn't plugged in (or if a teammate doesn't have a MIDI device) then that tuning is inaccessible.
* Sometimes I don't have my controller plugged in (or am at a different machine) and just want to quickly use an app that requires it.

So in order to address these downsides, I've made it easy to bind both to a knob/button and Dear ImGui widgets simultaneously. This gives it all the advantages of a hardware MIDI workflow, but with the fallback onto the already excellent ImGui workflow. ImGui thus handles numeric input and output, visualizes the colorpickers and other such things, for when that's useful.

## Goals
This is meant to be a simple 'drop in' library, specifically for developer MIDI usage described above. It's not meant to be a generic MIDI lib or support any arbitrary device, though code is designed such that it's easy to add your own devices by hand. In practice, there seems to be a handful of useful midi devices for this particular purpose anyway.

## Usage
The header should be pretty self explanatory. Functions with a 'twister' prefix use the Midi Fighter Twister knobs, and those with the 'fighter' prefix use the Midi Fighter 3D buttons. Some cool things to call out though:
* When using the basic 'twisterSliderFloat', clicking the knob will copy the value to clipboard. This is handy when you just want to quickly bind a knob to some magic constant to tune it, and then copy paste the result back into code. The click will also print to console, if you supplied a print function in **devmidi_custom.h**.
* Use `twisterSliderClickDefault` to set a value to a provided default when the knob is clicked.
* Use `twisterSliderClickToggle` to quickly jump between the provided min and max values (in addition to tuning between them).
* Use `twisterColorEdit` to edit colors. In HSV mode, the hue will wrap. 
* Use `fighterRadioButton` with multiple buttons to quickly toggle options.
* Use `fighterCheckboxMomentary` to enable a checkbox only when a button is held down. Useful for momentarily turning on a visualization to quickly check for problems, rather than toggling it on permanently.

## "Architecture" (it's just some functions really)
The library is actually in two pieces: the device specific API that is **devmidi.h/cpp**, and the underlying device agnostic **midi_wrap.h/cpp which** just listens on the devices and wrangles the output into a convenient format. Right now, it only listens for Continuous Controller and Note messages, since that's what I've found to be useful. This breakdown abstracts the bits of MIDI that I care about and makes it easier to add binds for new devices.
The devmidi layer is meant to be hacked on so you can quickly add your own particular device and functionality.
**devmidi_custom.h** is the place to provide your own print function for values, and also to define convenience wrappers for your own app specific vector structs (like vec3, float4, color3 that sort of thing).

## Building
It should 'just work' if you drop it into a Visual Studio project. You might have to adjust the path to your ImGui in the code, but that's probably it.
Right now, **midi_wrap** is Windows only, so while **devmidi** is platform agnostic, the whole thing only works on Windows. Presumably it shouldn't be hard to add Mac/Linux support to **midi_wrap**, since the MIDI protocol is the same, it's just the OS calls that need to change (and on Windows at least, it's like a half dozen API calls). I don't plan to do this work since I don't have machines to test on, but if this seems useful to you and you want to do it, I'm happy to help in spirit and discussion of what needs doing :)

## Future Work
* It would be cool to actually send color commands to the twister/fighter to reflect the app values.
* Both the fighter and twister have buttons to switch between 'banks' which would allow them to present 64 knobs/buttons instead of the current 16. This might be handy to support for larger apps, which might dedicate a given bank to a given task (like graphics or gameplay). In practice, it seems more useful to bank the actual data in software, but it would still be nice to use the hardware buttons to switch the banks.

## Which controller should I get?
I get asked sometimes which controllers are best for this sort of workflow.
I am not at all affiliated with Midi Fighter or DJ Tech Tools, but I've personally found their devices to be the best so far. I'm always on the lookout for new ones :)

Here are some criteria to consider:
* Absolute vs Relative Knobs - **This is the most important point:** I much prefer **endless** knobs that output **relative** values for this work. You almost certainly will want relative because you never know what value your app starts at, and you don't want to 'sync' your controller to those values. On the flipside, you don't want values you 'left' on your controller to be the 'defaults' for your app (the next day after tuning, you will forget that knob9 controlling brightness was left at 0.72 and be very confused that your screen seems 'a bit dark for some reason?'). You also don't want to have to deal with 'pickup' or 'syncing' for bounded knobs, best to just let it spin (the twister can be configured into absolute or relative, and has LED display for bounds, so in theory you can sync to them if you feel like it). Motorized knobs solve some of these issues, but they're extremely expensive for this niche usecase. Also with endless relative knobs you can easily control unbounded values like log-scale.
* Knob encoder precision - You want the best obviously, but this directly impacts price (the twister has good bang-for-buck encoders). Lower precision encoder means that you have to turn the knob a lot more to cover a broader range with the same accuracy, which can get really tedious.
* Knob click - This is very useful to 'reset' values to defaults, to print values, or to change precision. Can also be used as buttons. Clicking the knobs might change the values slightly (the code attempts to mitigate this, but it can still happen). If you want a totally 'rock solid' reset button, consider getting a controller with dedicated buttons next to each knob for that.
* Knob count - You probably will end up using at least 8 (colors take 3-4 each). I've maxed out 16 in some apps, but it's hard to remember what they all do past that.
* Knob arrangement - The 4x4 layout of the twister is nice and compact, and it's good for logical sets of 4 values (nice for colors). But it's worse for tuning an 8-knot easing spline (because you have to span two rows), for that a linear 1x8 or 1x16 layout is much more intuitive. 
* Knob/Button size - If you're doing this in VR, you might want them easy to find and press.

Knobs are far more useful than buttons, so if you're only going to get one, just get the knobs. If the knobs are clickable, you can use them as buttons too in a pinch, but they don't feel nearly as nice. Knob rotation can also be used to set radio buttons, but again it's not as nice as doing it with buttons. Buttons are mainly useful for debugging (visualize a buffer, fire off an action), whereas knobs are most useful for tuning floats, colors, vectors, integers etc.