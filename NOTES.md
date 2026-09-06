# NOTES — 8BitDo Retro R8 Mouse on macOS

The lab notebook. `README.md` says what to do; this says why, and what the hardware
will not let you do at all.

Status: **investigation, no rules written yet.** Everything about which buttons emit what
has been confirmed against real EventViewer captures in both hand modes. The few points
still marked *unverified* say so explicitly.

---

## What the mouse actually sends

The R8 is a composite device. It presents a pointing interface *and* a keyboard
interface, and the side buttons use the keyboard one. That single fact drives the whole
design.

| Button | Diagram piece | Reaches macOS as | Mappable in Karabiner |
|---|---|---|---|
| Primary (left click) | 1 | pointing button | not needed |
| Middle / wheel press | 2 | pointing button | yes |
| Secondary (right click) | 3 | pointing button | not needed |
| DPI switch | 5 | nothing, in either hand mode | no — see below |
| Button 4 | 6 | `left_option` + `close_bracket` | **yes** |
| Button 5 | 7 | `left_option` + `open_bracket` | **yes** |
| Button 6 | 8 | nothing; in left-hand mode, B4's codes | no — see below |
| Button 7 | 9 | nothing; in left-hand mode, B5's codes | no — see below |

The table describes the default right-hand mode. Left-hand mode does not add anything to
it, it only moves the two live buttons to the other flank — see below.

Buttons 4 and 5 producing `‘` and `“` is not a fault. On a US layout `Option`+`]` is `‘`
and `Option`+`[` is `“`; the mouse is sending the keystroke and macOS is rendering it
correctly. 8BitDo shipped a keystroke where most mice ship a HID back/forward button.

## Buttons 6 and 7 are not broken, and Karabiner cannot reach them

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

## So there is exactly one way to reach Buttons 6 and 7

**8BitDo Ultimate Software V2 for macOS.** Assign them in the mouse's own profile so they
emit a keystroke; once they emit something, Karabiner sees it and the normal rules apply.

A macOS build does exist, and the confusion is worth heading off because the Windows build
is the one people find first. It ships as `UltimateSoftwareV2.dmg`, requires macOS 13.0 or
above, and runs on both Intel and Apple Silicon. The macOS build supports a shorter device
list than the Windows one, but `Retro R8 Mouse N Edition` is on it by name.

Without it, this mouse offers two mappable thumb buttons and a mappable wheel press. Plan
around that rather than hunting for a trick; there isn't one.

## Two buttons need not mean two actions

Karabiner can get considerably more than two actions out of two buttons, which is the
practical answer to the shortfall above:

- `to_if_alone` and `to_if_held_down` split each button into a tap and a hold.
- A `simultaneous` rule on both buttons at once gives a third gesture.
- A button held as a layer modifier changes what the wheel or the other button does.

Five or six distinct actions from two switches is realistic. None of it requires the
vendor software.

## The DPI switch button

Cycles DPI (800 / 1200 / 1600 / 2400 / 3200 / 6400, signalled by indicator colour) and is
handled entirely in firmware without notifying the host. Confirmed silent in EventViewer
in both hand modes. Karabiner cannot repurpose it, and it falls into the same bucket as
Buttons 6 and 7 — Ultimate Software or nothing.

## The recommended architecture

Where a button is silent, use Ultimate Software for the *minimum* job: make it emit some
unique, otherwise-unused keystroke. Do the actual mapping in Karabiner.

This keeps the interesting logic in a file that can be read, versioned, and shared, and
leans on the vendor tool only for the one thing Karabiner cannot do. It also means the
mouse profile stays simple enough to rebuild from memory after a factory reset.

## Every rule needs `device_if`

The sibling keyboard project scopes its rules with `device_unless is_built_in_keyboard`.
This project needs the opposite and stricter form: `device_if` naming the mouse's own
vendor and product ID.

The reason is the keyboard interface. A rule matching `left_option` + `close_bracket`
with no device filter fires just as happily when that combination is typed on the real
keyboard, and quietly eats a character the user meant to produce. The filter is not
politeness here, it is correctness.

The mouse does not present one identity, it presents two, and a rule set naming only one
of them goes silent the moment you change how the mouse is connected:

| Device | Vendor ID | Product ID |
|---|---|---|
| `8BitDo Retro R8 Mouse` | `11720` | `20997` |
| `8BitDo Retro R8 Mouse Adapter` | `11720` | `20998` |

`device_if` accepts a list, so name both and stop thinking about it. Vendor `11720` is
8BitDo's, shared across their whole range — it is not specific enough on its own, and a
vendor-only filter would also catch an 8BitDo keyboard on the same machine.

Karabiner draws the mouse as a single row with a keyboard-over-mouse icon and the adapter
as two separate rows, one per interface. That is a display quirk, not a difference in how
the devices behave.

## Karabiner and pointing devices

Karabiner treats pointing devices separately from keyboards, and events from a mouse may
not appear in EventViewer until that device is ticked for modification in
`Karabiner-Elements → Devices`. If the wheel and middle click show nothing, check there
before concluding the hardware is silent. *Unverified* — this explains the observed
absence but has not been tested on this device.

## Sources

- 8BitDo Retro R8 Mouse manual, English pages 01–06. No page carries an edition name,
  but the PDF's embedded document title is `8BitDo-Retro-R8 Mouse-Xbox-Edition`. The
  button diagram and mode chords match the N Edition hardware.
- 8BitDo's announcement of macOS Ultimate Software V2 support for the N Edition.
