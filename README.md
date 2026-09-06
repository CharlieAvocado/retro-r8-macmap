# 8BitDo Retro R8 Mouse MacMap with Karabiner

A [Karabiner-Elements](https://karabiner-elements.pqrs.org/) rule set for the 8BitDo
Retro R8 Mouse on macOS. It gives the thumb buttons Back and Delete, adds forward-delete
on `Ctrl`, moves the wheel press to `Return`, and makes the two dead side buttons usable.

**Every rule is scoped to the mouse's own vendor and product IDs, so nothing else you type
is affected.** This matters more here than on a keyboard: the side buttons arrive as
*keyboard* events, and an unscoped rule would eat real keystrokes.

Two files do the work:

| File | What |
|---|---|
| `retror8-macmap.json` | The rule set. A bare `{description, manipulators}` object |
| `NOTES.md` | The lab notebook — what the hardware sends, why two of the buttons are silent out of the box, why left-hand mode is not the answer, and why each rule is shaped the way it is. Read it before changing anything |

---

## What you get

| Button | Becomes |
|---|---|
| Button 4 (front thumb) | Back |
| Button 5 (rear thumb) | Delete (backspace) |
| Button 5 + `Ctrl` | Delete forward |
| Wheel press | `Return` |
| Buttons 6 and 7 | Yours to assign — see below |

Left click, right click and wheel scroll are untouched. The DPI switch cannot be
repurposed by any means; `NOTES.md` explains why.

---

## Setup

You must have already installed Karabiner-Elements and Karabiner-EventViewer.

### 1. Enable the mouse in Karabiner

**`Karabiner-Elements → Devices` → tick "Modify events" for the Retro R8 Mouse.**

Karabiner ignores pointing devices by default. Until this is ticked nothing here fires,
and the mouse's events will not even appear in EventViewer — which looks exactly like
broken hardware. This is the most common reason a mouse rule set appears to do nothing.

The mouse appears more than once in the list, and under more than one identity depending
on how it is connected. Tick every row that is the R8.

### 2. Clear any Simple Modifications for this mouse

**`Karabiner-Elements → Simple Modifications` → remove any entries targeting the R8.**

Simple Modifications run *before* complex modifications, so a leftover entry silently
rewrites a button out from under these rules.

While debugging, know that **EventViewer shows events after Simple Modifications have been
applied** — a button with one on it displays as whatever it was rewritten to, not as what
the hardware sent.

### 3. Assign the side buttons in 8BitDo's app

This step is what makes Buttons 6 and 7 work at all. It is also what makes the rule set
predictable, so do it even if you only care about Buttons 4 and 5.

Download **Ultimate Software V2** from [app.8bitdo.com](https://app.8bitdo.com/) — the
vendor's own site, not a mirror. It needs macOS 13 or later. Connect the mouse **by USB
cable**; the app is unlikely to see it over Bluetooth.

Set each side button to `Function`, then:

| Button | Assign to |
|---|---|
| Button 4 | `F16` |
| Button 5 | `F17` |
| Button 6 | `F18` |
| Button 7 | `F19` |

Save the profile to the device. macOS binds nothing to `F16`–`F19` and no keyboard types
them by accident, which is exactly why they are the right choice — they are couriers, not
functions. Karabiner decides what they mean.

The profile is written into the mouse, so you can quit the app afterwards and the buttons
keep working. A factory reset wipes it.

**If you skip this step**, Buttons 4 and 5 still work — the rule set also carries rules
for the factory keystrokes — but Buttons 6 and 7 stay dead. See `NOTES.md`.

### 4. Import the rules

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

## Assigning Buttons 6 and 7

They are deliberately left unmapped. `F18` and `F19` do nothing on macOS, so the buttons
sit inert and harmless until you decide what you want — and the hard part, making them
emit anything at all, is already done in step 3.

To give one a job, add a manipulator like this to the list, changing only the `to` block:

```json
{
  "description": "B6 (F18) -> Mission Control",
  "from": {
    "key_code": "f18",
    "modifiers": { "optional": ["caps_lock"] }
  },
  "to": [{ "key_code": "mission_control" }],
  "type": "basic",
  "conditions": [
    {
      "type": "device_if",
      "identifiers": [
        { "vendor_id": 11720, "product_id": 20997 },
        { "vendor_id": 11720, "product_id": 20998 }
      ]
    }
  ]
}
```

Use `f19` for Button 7. Keep the `conditions` block exactly as it is — without it the rule
stops being scoped to the mouse.

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

Do not drop to a vendor-only filter. Vendor `11720` is 8BitDo's whole range, so it would
also catch an 8BitDo keyboard on the same machine.

---

## Changes from stock

Nothing here is a fix for a defect. The mouse works as designed on Windows; the design
just assumed a Windows layout. Out of the box Buttons 4 and 5 send `Option`+`]` and
`Option`+`[`, which on a US Mac layout type `‘` and `“` — the app's Forward and Back
functions, expressed in Windows shortcuts. These rules put them to better use, and give
you two buttons the mouse otherwise wastes entirely.
