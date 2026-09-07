# NOTES — 8BitDo Retro R8 Mouse on macOS

The lab notebook. `README.md` says what to do; this says why, and what the hardware
will not let you do at all.

Everything here has been confirmed on hardware: which buttons emit what, in both hand
modes, from real EventViewer captures, and every rule in `retro-r8-macmap.json` tested in
place. Nothing in this document is inferred unless it says so.

It records what the mouse does and what Karabiner can do about it. What the buttons are
mapped *to* is taste, and lives in the `README` — those choices could all be different
without a word here changing. The mouse is a playground; this is the map of it.

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

- Left-hand mode: hold `Button 6` + `Button 7` + `Middle` for 5 seconds.
- Right-hand mode: hold `Button 4` + `Button 5` + `Middle` for 5 seconds.

Each pair is the thumb pair for one hand. The mouse ships in right-hand mode, so the
left-flank pair (4 and 5) is live and the right-flank pair (6 and 7) is muted. The
firmware still reads 6 and 7 — it has to, or the left-hand chord could not work — but it
sends nothing to the host.

**Karabiner is an event transformer. It cannot invent an event that never arrives.** No
`device_if`, `simultaneous`, or `mandatory` construction reaches a button that is silent
on the wire. EventViewer showing nothing is the correct and complete answer, not a
symptom of a missing setting.

They are also not 8BitDo's Super Buttons. That feature is programmed from an 8BitDo
keyboard, and running the procedure with one present does not offer the mouse as a target.
Tested, not assumed.

## Left-hand mode is a mirror, not an unlock

The obvious free experiment is to switch modes and see whether the other pair wakes up.
It does. It is still not what you want, and this is confirmed by test rather than
inferred:

- In left-hand mode, `Button 6` and `Button 7` fire — but they emit `left_option` +
  `close_bracket` and `left_option` + `open_bracket`. Exactly what 4 and 5 emit in
  right-hand mode. They inherit the identities; they do not get their own.
- `Button 4` and `Button 5` go silent in exchange.
- Primary and secondary click swap too. You left-click with the right button.

The mode flips the entire mouse. **Out of the box you never have more than two thumb
buttons.** You only choose which flank they sit on, and pay for it with reversed clicks.
For a right-handed user there is nothing here. Getting all four takes the vendor app.

You cannot get stranded trying it. The return chord is read in firmware rather than over
HID, which is the same reason `Button 6` + `Button 7` can trigger the switch while sending
nothing to the host. Muted buttons still form chords.

## Reaching Buttons 6 and 7: the vendor app, used sparingly

**8BitDo Ultimate Software V2** is the only way to make them emit anything. A macOS build
exists. The Windows build is the one that surfaces first in most searches, which is the
usual source of the belief that there is no Mac version. It ships as
`UltimateSoftwareV2.dmg`, requires macOS 13 or above, runs on Intel and Apple Silicon, and
the R8 is on its supported list by name. Connect the mouse by cable — the app is unlikely
to find it over Bluetooth.

The app can assign a side button from several groups: alphanumeric, function, numpad,
navigation, modifiers, symbols, mouse functions, shortcuts, macros, or disable. The DPI
switch does not appear in that list at all, which seems the final word on repurposing it.

**Use the app for as little as possible.** Its only job here is to make a silent button
emit something unique; Karabiner decides what that means. Concretely, all four side buttons
are assigned to `F16` through `F19` in button order and nothing else is touched.

`F16`–`F19` because macOS binds nothing to them and no keyboard produces them by accident.
They are couriers, not functions. There is no useful assignment to find inside the app, and
none is needed.

The profile is written into the mouse rather than held by a running helper, so the app can
be quit and even uninstalled, and the buttons keep working.

A factory reset *will* wipe it, and the app has to be run again to put it back. It does not
break everything, though: the rule set also carries manipulators for the factory keystrokes,
so Buttons 4 and 5, the wheel press and forward-delete keep working on a reset mouse.
Buttons 6 and 7 are the only casualties, because a reset returns them to sending nothing.

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

Macros earn their place for long literal sequences to be typed identically everywhere. That
is not the use here.

## Writing a profile also rewrites Buttons 4 and 5

This is easy to miss, because it happens without being asked for.

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

Two consequences:

- **The app alone fixes this mouse for Mac.** Anyone who only wants working Back and
  Forward needs no Karabiner at all — install the app, save a profile, done.
- Assigning Buttons 4 and 5 explicitly is not busywork. It pins them to a state the app
  will not quietly change underneath you.

The rule set keeps manipulators for the factory keystrokes as well as the assigned ones.
They match different inputs so they cannot conflict, and they mean the set still works on
a mouse that has never met the app — or on one that has been factory reset.

## Getting more actions than you have buttons

The mouse offers five switches Karabiner can reach: Buttons 4 to 7 and the wheel press.
Each of those can carry more than one action, so five switches is not a ceiling of five
actions. Six mechanisms are available, and they differ in what they add and what they cost.
`builder.html` generates the first three plus chords; layers, application scope and
multi-tap are hand-written.

| Mechanism | Karabiner construct | Adds | Costs |
|---|---|---|---|
| Tap versus hold | `to_if_alone` with `to_if_held_down` | one action per switch | the action you already had moves from key-down to key-release, so it starts firing later |
| Modifier variants | `mandatory` in `from.modifiers` | one action per modifier per switch | needs the other hand on a keyboard |
| Simultaneous chord | `simultaneous` | one action per pair | each member's solo press must wait out the threshold, so the cost lands on gestures already in use. Must sit above its own buttons' rules |
| Layer button | `set_variable` with a `variable_if` condition | one action per target under the layer | the layer button gives up its own actions |
| Application scope | `frontmost_application_if` | multiplies every action above | one manipulator per application per action |
| Multi-tap | `to_delayed_action` with a counter variable | one or two per switch | the single tap cannot commit until the window expires |

A single switch, without involving any other:

```
  Button 6 ─┬─ tap ........................ action 1
            ├─ hold ....................... action 2
            ├─ Ctrl + press ............... action 3
            ├─ Opt + press ................ action 4
            └─ Cmd + press ................ action 5
```

Spending a button as a layer instead, which is what makes the wheel reachable:

```
  Button 7 held ─┬─ Button 4 ............... action A
   (as a layer)  ├─ Button 5 ............... action B
                 ├─ Button 6 ............... action C
                 └─ any keyboard key ....... action D, E, F ...

  Button 7 alone ─── nothing. The layer costs it every action of its own.
```

A layer is not limited to the mouse's own buttons. The variable it sets is visible to every
manipulator in the set, so holding a thumb button can change what the keyboard does too.

**What it actually comes to.** The rule set as shipped, extended through the builder,
reaches **36 distinct triggers**: five switches each with a bare press, four single-modifier
variants and a hold, plus the six two-button chords.

The ceiling is far higher and mostly beside the point. `from.modifiers.mandatory` accepts
side-specific names and must match exactly, so `left_shift` and `right_shift` are two
separate triggers — eight physical modifiers give 256 states. `simultaneous` accepts three
and four members, not just pairs, so the four side buttons yield fifteen non-empty subsets
plus the wheel alone. At 16 trigger groups × 256 modifier states × 2 for tap and hold, the
arithmetic says **8,192**, and counting `fn` as a ninth modifier doubles it.

**That number is useless, and it is worth knowing why.** Karabiner walks the manipulator
list top to bottom on every event and takes the first match, so each of those 8,192
triggers is a literal manipulator. Perceptible input lag is reported in the low thousands —
an order of magnitude below the ceiling. Most of the 256 modifier states are already
claimed by macOS or the frontmost application. And a hand holds five to nine gestures per
switch, which is roughly what the builder generates.

So the gap between 36 and 8,192 is not a missing feature. **The builder is sized to the
operator rather than to the tool**, deliberately.

Two things the arithmetic cannot see. **Adjacency:** the two left-flank buttons are worked
by the same thumb, so a chord across them is awkward in a way no rule can express, and
spending one as a layer usually beats chording it. **Vocabulary:** triggers are worthless
without actions to put in them, and widening the action list costs no gesture at all — it
is the cheapest gain available and should be exhausted before reaching for a new mechanism.

**Where this started.** Out of the box, across those five switches, exactly one did
something useful: the wheel press was a middle click. Buttons 6 and 7 sent nothing at all.
Buttons 4 and 5 typed `‘` and `“` into whatever had focus, which is worse than nothing. The
DPI switch was and remains unreachable. One useful action became thirty-six.

**Why a single headline number is misleading.** The mechanisms compete for the same
presses. A button spent as a layer modifier no longer has a tap or a hold. A pair committed
to a chord makes each of its members slower to fire alone, because Karabiner must wait to
see whether the partner is coming. Multiplying six mechanisms by five switches gives a
number that cannot all be true at once.

The arithmetic that does hold: tap and hold across the four side buttons is eight, the wheel
press makes nine, and one modifier variant across the four side buttons brings it to
thirteen — with no layer and no chord spent yet. Adding a second modifier, the chord pairs
and a layer takes it well past twenty.

There is no ceiling worth respecting here. A DAW user mapping transport and track controls
in Logic Pro, or anyone driving an application with a large fixed command set, has a real
reason to want every gesture the hardware can express. Build as many as are useful; the only
genuine cost is the hand having to remember them.

Application scope is the mechanism worth reaching for first, because it is the only one that
costs no gesture at all. The same press does one thing in a browser and another in Logic,
and the hand never learns anything new. For a large command set this is usually the answer:
one modest set of gestures, redefined per application.

None of this needs the vendor app. It is all in the rule set.

### Why not the D-pad model

8BitDo sells a second NES-styled mouse, the **N30 Wireless Mouse**, which puts a D-pad on
the left flank where the R8 has its two thumb buttons. For a project like this the R8 is
the better base, for two reasons.

The N30's D-pad is reported to be fixed rather than programmable — left and right for
browser back and forward, up and down for page scrolling. Fixed functions leave nothing to
catch: no assignable keystroke means no `from` value, and the same wall as the R8's DPI
switch. *Unverified* — taken from reviews rather than tested.

The deeper reason survives even if that turns out to be wrong. A D-pad's directions are
mutually exclusive. Up and down cannot be held together, so the chord and layer mechanisms
in the table above mostly evaporate: four directions give four actions plus taps and holds,
and little else. Four independent switches give six chordable pairs on top of their own
taps and holds, and any one of them can be spent as a layer. Independent switches multiply;
a D-pad's directions only add.

## Editions, and why the difference does not matter

The R8 ships in several colourways — N Edition, Xbox Edition, C64 Edition, Forest — and
8BitDo describes them as functionally identical, differing in colour and nothing else. Every
one has the PAW 3395 sensor, four programmable side buttons, the same three connection
modes, and the same charging dock. Nothing in this rule set is edition-specific.

The C64 Edition, added in March 2026, is a tan shell with a dark brown wheel, sides and
underside, and red side buttons. The N Edition is off-white with grey sides and red side
buttons. The buttons sit in the same places and send the same things.

**Do not confuse the R8 with the N30 Wireless Mouse**, which is the D-pad model and a
different product entirely — see the comparison further up.

If your device identifiers differ from the two the rule set names, that is a firmware or
connection difference rather than an edition difference. The `README` covers what to do.

## What the scroll wheel can and cannot do

The wheel press is an ordinary `pointing_button` and is fully mappable. **Scroll direction
is not.** Karabiner has no way to trigger on a scroll event, so there is no rule that reads
"scroll up does X". `pointing_button` runs `button1` through `button255` and none of them is
a wheel direction; the feature request for it was closed without being implemented, with the
maintainers stating plainly that no option exists to get scroll direction.

What is available sits outside the manipulator system, in `Karabiner-Elements → Devices`
under the mouse's own settings:

- `Flip mouse vertical wheel` and `Flip mouse horizontal wheel`, to reverse a direction.
- `Swap mouse wheels`, to exchange vertical and horizontal.
- `Wheels multiplier`, to change scroll speed for this mouse alone without touching the
  trackpad. Worth knowing if macOS scroll speed is set globally and the mouse wants
  something different.

There is also `mouse_motion_to_scroll`, which turns cursor movement into scrolling while a
key is held. That is the reverse of what most people want here, but it is the one place
where scrolling and the rule set meet.

## The DPI switch button

Cycles DPI (800 / 1200 / 1600 / 2400 / 3200 / 6400, signalled by indicator colour) entirely
in firmware, telling the host nothing. Confirmed silent in EventViewer in both hand modes,
and absent from the app's list of assignable buttons. No route to it could be found, which
is not the same as proving there is none.

The levels themselves *can* be edited in the app, which allows any six values. The nearest
thing to switching the button off is setting all six the same, or separating them by a
negligible amount. The button still cycles, but cycling stops meaning anything.

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
`F13`–`F16` but never *match* on them, so nothing collides today.

That is confirmed by test rather than read off the JSON. The keyboard's `Insert` becomes
`F16` through a complex modification, and the mouse's Button 4 emits `F16` directly, yet
pressing `Insert` does not trigger Back. Two independent reasons: Karabiner does not feed a
manipulator's output back through the chain, and the condition names the mouse anyway.

What would break it is a rule with `f16` through `f19` in its `from` being added to the
keyboard project — this mouse would start firing it. The fix then is to add a
`device_unless` for the mouse's IDs there, not to move this project off the F-keys.

In `Karabiner-Elements → Devices`, Karabiner draws the mouse as a single row with a
keyboard-over-mouse icon and the adapter as two rows, one per interface. The list also
changes as devices come and go. That is display, not behaviour — do not read meaning into
which icons appear.

## Rule order, and why it barely matters here

The forward-delete rule sits above the plain Delete rule. Karabiner takes the first
manipulator that matches, so a broad rule above a narrow one hides it.

Here the ordering is belt-and-braces rather than load-bearing. `mandatory` means *exactly
these modifiers*, so a bare `F17` and `Ctrl`+`F17` cannot both match the same manipulator;
either order would work. It is written this way so that adding an `optional` clause later —
which would make the plain rule greedy — cannot silently break forward-delete. Keep
specific above general and the trap never opens.

`optional: ["caps_lock"]` appears on every rule. Without it, having caps lock on stops every
rule from matching, which is hard to diagnose.

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
Both halves matter:

- A button carrying a Simple Modification displays as its replacement, not as what the
  hardware sent. At this stage a rewrite made in Simple Modifications is indistinguishable
  from firmware behaviour.
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

EventViewer cannot diagnose this. It reports `button3` correctly whether the rule fires or
not, because it sits upstream of complex modifications. The symptom is behavioural only, and
the fix is a checkbox rather than a change to any rule.

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

- 8BitDo Retro R8 Mouse manual, English pages 01–06. No page carries an edition name, and
  the file's embedded document title reads `8BitDo-Retro-R8 Mouse-Xbox-Edition` — most
  likely just a naming slip in the file itself, since the booklet's own pages describe the
  mouse generically. It applies to any of them regardless; see below.
- 8BitDo's announcement of macOS Ultimate Software V2 support for the Retro R8 Mouse.
- Karabiner-Elements documentation, *Choose devices*: "Mice are disabled by default. You
  have to enable them if you want to change the mouse buttons in Karabiner-Elements."
- Karabiner-Elements `NEWS.md`, for `ignore_vendor_events`.
- Reviews of the 8BitDo N30 Wireless Mouse, for its D-pad's fixed functions. Not tested
  first-hand.
