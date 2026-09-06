# NOTES — 8BitDo Retro R8 Mouse on macOS

The lab notebook. `README.md` says what to do; this says why, and what the hardware
will not let you do at all.

Everything here has been confirmed on hardware: which buttons emit what, in both hand
modes, from real EventViewer captures, and every rule in `retror8-macmap.json` tested in
place. Nothing in this document is inferred unless it says so.

---

## What the mouse actually sends

The R8 is a composite device. It presents a pointing interface *and* a keyboard
interface, and the side buttons use the keyboard one. That single fact drives the whole
design.

| Button | Diagram piece | Factory firmware sends | After the app | Mappable |
|---|---|---|---|---|
| Primary (left click) | 1 | pointing button | unchanged | not needed |
| Middle / wheel press | 2 | `pointing_button` `button3` | unchanged | yes |
| Secondary (right click) | 3 | pointing button | unchanged | not needed |
| DPI switch | 5 | nothing | nothing | never |
| Button 4 | 6 | `left_option` + `close_bracket` | `f16` | yes |
| Button 5 | 7 | `left_option` + `open_bracket` | `f17` | yes |
| Button 6 | 8 | nothing | `f18` | only after the app |
| Button 7 | 9 | nothing | `f19` | only after the app |

The factory column describes the default right-hand mode. Left-hand mode adds nothing, it
only moves the two live buttons to the other flank — see below. The `f16`–`f19` column is
the assignment this project settled on, not something the app does by itself; the reasoning
is further down.

Buttons 4 and 5 producing `‘` and `“` is not a fault. On a US layout `Option`+`]` is `‘`
and `Option`+`[` is `“`; the mouse is sending the keystroke and macOS is rendering it
correctly. 8BitDo shipped a keystroke where most mice ship a HID back/forward button.

## Why Buttons 6 and 7 are silent out of the box

The R8 has two side buttons on each flank. The manual's mode chords give the design away:

- Left-hand mode: hold `Button 6` + `Button 7` + Middle for 5 seconds.
- Right-hand mode: hold `Button 4` + `Button 5` + Middle for 5 seconds.

Each pair is the thumb pair for one hand. The mouse ships in right-hand mode, so the
left-flank pair (4 and 5) is live and the right-flank pair (6 and 7) is muted. The
firmware still reads 6 and 7 — it has to, or the left-hand chord could not work — but it
sends nothing to the host.

**Karabiner is an event transformer. It cannot invent an event that never arrives.** No
`device_if`, `simultaneous`, or `mandatory` construction reaches a button that is silent
on the wire. EventViewer showing nothing is the correct and complete answer, not a
symptom of a missing setting.

They are not "super buttons." The manual describes no such feature, and nothing in the
device's behaviour suggests one.

## Left-hand mode is a mirror, not an unlock

The obvious free experiment is to switch modes and see whether the other pair wakes up.
It does. It is still not what you want, and this is confirmed by test rather than
inferred:

- In left-hand mode, `Button 6` and `Button 7` fire — but they emit `left_option` +
  `close_bracket` and `left_option` + `open_bracket`. Exactly what 4 and 5 emit in
  right-hand mode. They inherit the identities; they do not get their own.
- `Button 4` and `Button 5` go silent in exchange.
- Primary and secondary click swap too. You left-click with the right button.

The mode flips the entire mouse. **You never have more than two thumb buttons.** You only
choose which flank they sit on, and pay for it with reversed clicks. For a right-handed
user there is nothing here.

You cannot get stranded trying it. The return chord is read in firmware rather than over
HID, which is the same reason `Button 6` + `Button 7` can trigger the switch while sending
nothing to the host. Muted buttons still form chords.

## Reaching Buttons 6 and 7: the vendor app, used sparingly

**8BitDo Ultimate Software V2** is the only way to make them emit anything. A macOS build
exists, which is worth stating plainly because the Windows build is the one people find
first. It ships as `UltimateSoftwareV2.dmg`, requires macOS 13 or above, runs on Intel and
Apple Silicon, and the R8 is on its supported list by name. Connect the mouse by cable —
the app is unlikely to find it over Bluetooth.

The app can assign a side button from several groups: alphanumeric, function, numpad,
navigation, modifiers, symbols, mouse functions, shortcuts, macros, or disable. The DPI
switch does not appear in that list at all, which is the final word on repurposing it.

**Use the app for as little as possible.** Its only job here is to make a silent button
emit something unique; Karabiner decides what that means. Concretely, all four side buttons
are assigned to `F16` through `F19` in button order and nothing else is touched.

`F16`–`F19` because macOS binds nothing to them and no keyboard produces them by accident.
They are couriers, not functions. Looking for a *useful* assignment inside the app is the
wrong instinct — there isn't one, and you don't want one.

The profile is written into the mouse rather than held by a running helper, so the app can
be quit, and the buttons keep working. A factory reset wipes it.

## Why not the app's macro feature

The app will run macros on these buttons, which looks like it could replace Karabiner
entirely. It cannot, and taking it would cost:

- **Shareability.** A macro lives in the mouse's firmware. It cannot be versioned, diffed
  or read, and it cannot be copied out of this repository — which would defeat the point
  of publishing anything.
- **Conditionality.** A macro is a fixed keystroke sequence. Karabiner does per-application
  behaviour, tap versus hold, both-buttons-at-once, and shell commands.
- **Durability.** A factory reset wipes macros. A rule set is a file.
- **Diagnosis.** A misbehaving macro is opaque. Karabiner shows every step in EventViewer.

Macros earn their place for long literal sequences you want typed identically everywhere.
That is not this.

## Writing a profile also rewrites Buttons 4 and 5

This is the trap, and it is easy to miss because it happens without being asked for.

Out of the box the app's Forward and Back functions on Buttons 4 and 5 are expressed as
`Option`+`]` and `Option`+`[` — the Windows shortcuts, which on a US Mac layout type `‘`
and `“`. That is the whole "not designed for Macs" story in one line.

Save any profile from the macOS app and those two buttons are silently rewritten to
`Command`+`]` and `Command`+`[` — the *macOS* Forward and Back. So there are three possible
firmware states, and a rule set written against one is dead against the others:

| State | Button 4 sends | Button 5 sends |
|---|---|---|
| Factory | `Opt`+`]` | `Opt`+`[` |
| App defaults, after any save | `Cmd`+`]` | `Cmd`+`[` |
| Explicitly assigned | `f16` | `f17` |

Two consequences worth carrying:

- **The app alone fixes this mouse for Mac.** Anyone who only wants working Back and
  Forward needs no Karabiner at all — install the app, save a profile, done.
- Assigning Buttons 4 and 5 explicitly is not busywork. It pins them to a state the app
  will not quietly change underneath you.

The rule set keeps manipulators for the factory keystrokes as well as the assigned ones.
They match different inputs so they cannot conflict, and they mean the set still works on
a mouse that has never met the app — or on one that has been factory reset.

## Getting more actions than you have buttons

Karabiner can split one button into several gestures, which is worth knowing before
concluding you have run out of buttons:

- `to_if_alone` and `to_if_held_down` split a button into a tap and a hold.
- A `simultaneous` rule on two buttons gives a gesture neither has alone.
- A button held as a layer modifier changes what the wheel or another button does.

None of it needs the vendor app, and it is the honest answer to wanting a fifth and sixth
action out of four switches.

## The DPI switch button

Cycles DPI (800 / 1200 / 1600 / 2400 / 3200 / 6400, signalled by indicator colour) entirely
in firmware, telling the host nothing. Confirmed silent in EventViewer in both hand modes,
and absent from the app's list of assignable buttons. There is no route to it.

The levels themselves can be edited in the app, so the nearest thing to a fix is collapsing
the cycle until pressing the button stops mattering.

## Why `device_if` and not `device_unless`

The sibling keyboard project scopes its rules with `device_unless is_built_in_keyboard`.
That is too loose here. This project needs the strict form, naming the mouse's own IDs.

The reason is the keyboard interface. A rule matching `left_option` + `close_bracket` with
no device filter fires just as happily when that combination is typed on the real
keyboard, and quietly eats a character the user meant to produce. `device_unless
is_built_in_keyboard` would still let the Retro 108 trigger it. The filter is not
politeness here, it is correctness, and it has to name the device rather than exclude one.

The `README` carries the identifiers and what to do if yours differ.

**The hazard runs the other way too, and it is not hypothetical.** The sibling keyboard
project scopes all of its rules with `device_unless is_built_in_keyboard`, which excludes a
laptop keyboard and nothing else — this mouse included. Those rules currently produce
`F13`–`F16` but never *match* on them, so nothing collides today. The moment a rule with
`f16` through `f19` in its `from` is added to that project, this mouse would start firing
it. If that ever happens, the fix is to add a `device_unless` for the mouse's IDs there,
not to move this project off the F-keys.

Karabiner draws the mouse as a single row with a keyboard-over-mouse icon and the adapter
as two rows, one per interface. The list also changes as devices come and go. That is
display, not behaviour — do not read meaning into which icons appear.

## Rule order, and why it barely matters here

The forward-delete rule sits above the plain Delete rule. Karabiner takes the first
manipulator that matches, so a broad rule above a narrow one hides it.

Here the ordering is belt-and-braces rather than load-bearing. `mandatory` means *exactly
these modifiers*, so a bare `F17` and `Ctrl`+`F17` cannot both match the same manipulator;
either order would work. It is written this way so that adding an `optional` clause later —
which would make the plain rule greedy — cannot silently break forward-delete. Keep
specific above general and the trap never opens.

`optional: ["caps_lock"]` appears on every rule. Without it, having caps lock on stops every
rule from matching, which is a miserable thing to diagnose.

Forward-delete asks for `control` rather than `left_control`, so either control key works.

Karabiner merges modifier flags across devices, so the `Ctrl` may be held on any keyboard
while the button press arrives from the mouse — the same way holding shift on one keyboard
capitalises a letter typed on another. Tested, not assumed; it is the one part of this set
whose behaviour was not obvious in advance.

`Ctrl` rather than `Cmd` is deliberate. On macOS the control key is where text-editing
bindings live — `Ctrl`+`D` is already forward-delete in most text fields — while `Cmd`
belongs to application commands, and `Cmd`+`Delete` already means delete to the start of the
line. Forward-delete is a text-editing operation, so it goes on the text-editing modifier.
Neither combination collides with anything, so this is about fitting the platform's habits
rather than avoiding a conflict.

## Where EventViewer sits in the chain

EventViewer reports events **after Simple Modifications and before Complex Modifications.**
Both halves of that have caught us out:

- A button carrying a Simple Modification displays as its replacement, not as what the
  hardware sent. Chasing a firmware explanation for a rewrite you made yourself is a good
  way to lose an hour.
- Nothing in this rule set ever changes what EventViewer prints. A working rule looks
  identical to a broken one there. Judge these rules by what the Mac does.

The same applies across projects: the sibling keyboard's `Insert` shows as `insert` in
EventViewer even though a complex modification turns it into `F16` downstream.

## Karabiner lists the two interfaces separately

The R8 is composite, and Karabiner gives each interface its own row on the Devices tab with
its own `Modify events` checkbox — the pointing interface carrying the main buttons and the
wheel, the keyboard interface carrying the side buttons. The vendor app splits them the same
way: buttons 1 to 3 are edited under one profile and 4 to 7 under another.

**Ticking one row is not ticking the other.** The failure this produces is confusing rather
than obvious: the side-button rules all work, and only the wheel press does nothing, which
reads like a bad rule instead of a device that was never enabled.

## Settings on the Devices tab that look relevant and are not

`Ignore vendor events` and `Manipulate caps lock LED` default differently across rows, and
the inconsistency invites fiddling. Leave both alone.

Vendor events are HID events on a manufacturer's private usage pages, outside the standard
keyboard and button pages. Nothing in this rule set — or in the sibling keyboard project —
reads one; every manipulator uses ordinary `key_code` and `pointing_button` values. The
setting cannot affect a rule that never looks at a vendor usage. Caps lock LED control is
meaningless on a mouse.

A setting that is not implicated is not worth changing. Changing one is how you acquire a
mystery.

## Sources

- 8BitDo Retro R8 Mouse manual, English pages 01–06. No page carries an edition name,
  but the PDF's embedded document title is `8BitDo-Retro-R8 Mouse-Xbox-Edition`. The
  button diagram and mode chords match the N Edition hardware.
- 8BitDo's announcement of macOS Ultimate Software V2 support for the N Edition.
- Karabiner-Elements documentation, *Choose devices*: "Mice are disabled by default. You
  have to enable them if you want to change the mouse buttons in Karabiner-Elements."
- Karabiner-Elements `NEWS.md`, for `ignore_vendor_events`.
