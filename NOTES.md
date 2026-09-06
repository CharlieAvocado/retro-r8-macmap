# NOTES — 8BitDo Retro R8 Mouse on macOS

The lab notebook. `README.md` says what to do; this says why, and what the hardware
will not let you do at all.

Status: **investigation, no rules written yet.** Sections marked *unverified* have not
been confirmed against a real capture.

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
| DPI switch | 5 | *nothing observed* | no — see below |
| Button 4 | 6 | `left_option` + `close_bracket` | **yes** |
| Button 5 | 7 | `left_option` + `open_bracket` | **yes** |
| Button 6 | 8 | *nothing observed* | no — see below |
| Button 7 | 9 | *nothing observed* | no — see below |

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

Two ways out, and only two:

1. **8BitDo Ultimate Software V2 for macOS.** Assign Buttons 6 and 7 in the mouse's own
   profile so they emit a keystroke. Once they emit something, Karabiner sees it and the
   normal rules apply.

   A macOS build does exist, and the confusion is worth heading off because the Windows
   build is the one people find first. It ships as `UltimateSoftwareV2.dmg`, requires
   macOS 13.0 or above, and runs on both Intel and Apple Silicon. The macOS build
   supports a shorter device list than the Windows one, but `Retro R8 Mouse N Edition`
   is on it by name.
2. **Switch to left-hand mode.** This wakes 6 and 7. Whether it mutes 4 and 5 in
   exchange — making it a swap rather than a gain — is *unverified*; the swap is inferred
   from the symmetry of the two chords, not from any statement in the manual. Worth
   testing before reaching for the vendor software, because it costs nothing.

   You cannot get stuck in left-hand mode. The return chord is read in firmware rather
   than over HID, which is the same reason `Button 6` + `Button 7` can trigger the switch
   while sending nothing to the host. Muted buttons still form chords.

## The DPI switch button

Cycles DPI (800 / 1200 / 1600 / 2400 / 3200 / 6400, signalled by indicator colour) and
is believed to be handled entirely in firmware without notifying the host. If EventViewer
shows nothing for it, Karabiner cannot repurpose it either, and it falls into the same
bucket as Buttons 6 and 7 — Ultimate Software or nothing. *Unverified* pending a capture.

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

Note that the vendor/product ID may differ between the three connection modes — 2.4 GHz
adapter, Bluetooth, and wired. A rule set built while docked over Bluetooth may go silent
when the adapter is plugged in. *Unverified*; confirm in EventViewer's Devices tab for
each mode actually used.

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
