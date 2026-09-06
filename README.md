# 8BitDo Retro R8 Mouse MacMap with Karabiner

A [Karabiner-Elements](https://karabiner-elements.pqrs.org/) rule set for the 8BitDo
Retro R8 Mouse on macOS. It turns the two thumb buttons into Back and Delete, adds a
forward-delete on `Ctrl`, and moves the wheel press to `Return`.

**Every rule is scoped to the mouse's own vendor and product IDs, so nothing else you
type is affected.** This matters more here than on a keyboard: the thumb buttons arrive
as *keyboard* events, and an unscoped rule would eat `Option`+`[` typed on your real
keyboard.

Two files do the work:

| File | What |
|---|---|
| `retror8-macmap.json` | The rule set. A bare `{description, manipulators}` object |
| `NOTES.md` | The lab notebook — what the hardware sends, why Buttons 6 and 7 cannot be reached, why left-hand mode is not the answer, and what each rule is shaped the way it is. Read it before changing anything |

---

## What you get

| Button | Was | Becomes |
|---|---|---|
| Button 4 (front-left thumb) | `Option`+`]`, typing `‘` | Back |
| Button 5 (rear-left thumb) | `Option`+`[`, typing `“` | Delete (backspace) |
| Button 5 + `Ctrl` | — | Delete forward |
| Wheel press | Middle click | `Return` |

Left click, right click, wheel scroll and the DPI switch are untouched.

**Buttons 6 and 7 are not in this table and cannot be.** They send nothing to macOS.
`NOTES.md` explains why, and why left-hand mode does not get you around it.

---

## Setup

You must have already installed Karabiner-Elements and Karabiner-EventViewer. Then two
things have to be right before the rules will do anything.

### 1. Enable the mouse in Karabiner

**`Karabiner-Elements → Devices` → tick "Modify events" for the Retro R8 Mouse.**

Karabiner ignores pointing devices by default. Until this is ticked, none of these rules
fire, and the mouse's events will not even appear in EventViewer — which looks exactly
like broken hardware. This is the single most common reason a mouse rule set appears to
do nothing.

The mouse may appear more than once in the list, and under more than one identity
depending on how it is connected. Tick every row that is the R8.

### 2. Clear any Simple Modifications for this mouse

**`Karabiner-Elements → Simple Modifications` → remove any entries targeting the R8.**

Simple Modifications are applied *before* complex modifications, so a leftover entry
silently rewrites a button out from under these rules. The wheel-press rule in particular
will look like it is being ignored, because by the time it runs the button is already
something else.

This also has a consequence worth knowing while debugging: **EventViewer shows events
after Simple Modifications have been applied.** A button with a Simple Modification on it
displays as whatever it was rewritten to, not as what the hardware sent.

### 3. Import the rules

1. Open `retror8-macmap.json` and copy the whole file.
2. `Karabiner-Elements → Complex Modifications`.
3. **Add your own rule** → paste → Add (or Save).

**To update it later**, edit that entry in place with its edit button, or delete it and
add a fresh one. Karabiner runs the copy you pasted in, so changing the JSON elsewhere
does not reach it.

If a rule does not seem to fire, check what Karabiner actually has loaded:

```bash
python3 -c "
import json,os
d=json.load(open(os.path.expanduser('~/.config/karabiner/karabiner.json')))
for p in d['profiles']:
    if not p.get('selected'): continue
    for r in p['complex_modifications']['rules']:
        for i,m in enumerate(r['manipulators']):
            print(i, m.get('description'))
"
```

---

## If your device IDs differ

The rules name two identities:

| Device | Vendor ID | Product ID |
|---|---|---|
| `8BitDo Retro R8 Mouse` | `11720` | `20997` |
| `8BitDo Retro R8 Mouse Adapter` | `11720` | `20998` |

Between them these cover Bluetooth and the 2.4 GHz adapter. If yours reports something
else — a different edition, or a firmware revision that enumerates differently — read the
values off `Karabiner-EventViewer → Devices` and edit the `device_if` blocks. There is one
in every manipulator; change them all or the set will behave inconsistently.

Do not be tempted to drop to a vendor-only filter. Vendor `11720` is 8BitDo's whole
range, so it would also catch an 8BitDo keyboard on the same machine.

---

## Changes from stock

Nothing here is a fix for a defect. The mouse works as designed on Windows; the design
just assumes a Windows layout. Buttons 4 and 5 ship as `Option`+`]` and `Option`+`[`,
which on a US Mac layout type `‘` and `“` — a keystroke where most mice send a HID
back/forward button. These rules put them to better use.
