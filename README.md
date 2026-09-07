# 8BitDo Retro R8 Mouse MacMap with Karabiner

A [Karabiner-Elements](https://karabiner-elements.pqrs.org/) rule set for the 8BitDo
Retro R8 Mouse on macOS. It makes all four side buttons reachable, including the two that
send nothing out of the box, and scopes every rule to the mouse alone.

**The button assignments are one person's taste and are meant to be changed.** They are the
easiest part of this to replace — see
[Changing what the buttons do](#changing-what-the-buttons-do). What is worth keeping is the
rest: the device scoping, the profile that wakes Buttons 6 and 7, and what `NOTES.md`
records about a mouse that is far stranger than it looks.

**Every rule is scoped to the mouse's own vendor and product IDs, so nothing else you type
is affected.** This matters more here than on a keyboard: the side buttons arrive as
*keyboard* events, and an unscoped rule would eat real keystrokes.

The files:

| File | What |
|---|---|
| `INDEX.md` | A map of everything here, and where to start |
| `retror8-macmap.json` | The rule set. A bare `{description, manipulators}` object |
| `RECIPES.md` | Ready-to-paste actions, and a worksheet for planning your own layout |
| `builder.html` | An offline page that assembles the rule set from your choices. Opens preloaded with the configuration above |
| `Retro-R8-Mouse-manual.pdf` | 8BitDo's booklet, for the button diagram and the mode chords |
| `NOTES.md` | The lab notebook — what the hardware sends, why two of the buttons are silent out of the box, why left-hand mode is not the answer, and why each rule is shaped the way it is. Read it before changing anything |

---

## What it does out of the box

| Button | Becomes |
|---|---|
| Button 4 (front thumb) | Back |
| Button 5 (rear thumb) | Delete (backspace) |
| Button 5 + `Ctrl` | Delete forward |
| Wheel press | `Return` |
| Buttons 6 and 7 | unassigned |

Back and Delete on the thumb buttons is a preference, not a recommendation. Plenty of
people would rather have Back and Forward, or page up and page down, or copy and paste.
Nothing about the rule set depends on the choice, and swapping it is a one-line edit.

Left click, right click and wheel scroll are untouched. The DPI switch cannot be
repurposed at all; `NOTES.md` explains why, and what to do about it instead.

---

## Setup

You must have already installed Karabiner-Elements and Karabiner-EventViewer.

Paths below are written as `Karabiner-Elements → <panel>`. In the app those panels sit in
the sidebar under a `Configurations` header, which is a heading rather than something to
click — so there is no extra step, only a place to look.

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

Judge these rules by what the Mac does, not by what EventViewer shows — it sits at a point
in the chain where a working rule and a broken one look identical. `NOTES.md` explains
where.

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

Save the profile to the device. `NOTES.md` explains why those four keys specifically, and
why nothing more should be changed in the app.

The profile is written into the mouse, so the app can be quit or even uninstalled afterwards
and the buttons keep working. A factory reset wipes it, and `NOTES.md` covers what survives
one.

**If you skip this step**, Buttons 4 and 5 still work — the rule set also carries rules
for the factory keystrokes — but Buttons 6 and 7 stay dead. See `NOTES.md`.

### 4. Import the rules

1. Open `retror8-macmap.json` and copy the whole file.
2. `Karabiner-Elements → Complex Modifications`.
3. **Add your own rule** → delete the default contents → paste the copied JSON → Add
   (or Save).

**To update it later**, edit that entry in place with its edit button, or delete it and
add a fresh one. Karabiner runs only the copy you paste in, so changing the JSON elsewhere
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

## Changing what the buttons do

Every button here is a one-line edit away from doing something else. Find the manipulator
by its `description`, and change only its `to` block.

| Button | Manipulator to edit |
|---|---|
| Button 4 | `B4 (F16) -> Back` |
| Button 5 | `B5 (F17) -> Delete` |
| Button 5 + `Ctrl` | `B5 + Ctrl (F17) -> Delete Forward` |
| Wheel press | `Middle button / wheel press -> Return` |
| Buttons 6 and 7 | not present yet — see below |

Two places to get the replacement from:

- `RECIPES.md` — around fifty ready-to-paste blocks, each in its own copy box: navigation,
  editing, tabs, spaces, media, shell commands, tap-versus-hold, and per-application rules.
  It also carries a worksheet listing every input the mouse can offer, with room to record
  what you want from each.
- `builder.html` — the same thing interactively. Open it in a browser, pick an action for
  each button, and it assembles the complete file. It starts loaded with the configuration
  above rather than empty, so you are editing a working set. No installation, and nothing
  leaves the page.

  **GitHub will not render it in the browser** — it shows HTML files as source. Either
  download the file and open it locally, which works offline, or enable GitHub Pages for
  the repository (`Settings → Pages`, source: deploy from the default branch) and it goes
  live at `https://<owner>.github.io/<repo>/builder.html`.

**If you change Button 4 or Button 5, change its `FACTORY FIRMWARE` twin too.** Those three
manipulators exist so the set still works on a mouse that has never met the vendor app, or
has been factory reset. Editing only one half gives a mouse that behaves differently
depending on which firmware state it is in — a genuinely confusing bug to chase later.

### Buttons 6 and 7

They ship unmapped on purpose. `F18` and `F19` do nothing on macOS, so the buttons sit
inert until you decide — and the hard part, making them emit anything at all, is already
done in step 3.

To give one a job, add a manipulator like this, changing only the `to` block:

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
stops being scoped to the mouse. These two need no factory twin, because a mouse without
the vendor profile has nothing to send.

`NOTES.md` covers the ways to get more than one action out of a single button — tap versus
hold, chords, layers, and per-application behaviour.

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
