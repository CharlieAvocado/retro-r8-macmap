# Index

Everything in this repository, what it is for, and where to start.

---

## Start here

| If you want to… | Go to |
|---|---|
| Get the mouse working | `README.md`, the Setup section — four steps, in order |
| Choose different actions for the buttons | `builder.html`, or `RECIPES.md` to paste by hand |
| Plan a layout before touching anything | The worksheet at the top of `RECIPES.md` |
| Work out why a rule is not firing | `README.md`, Setup steps 1 and 2. Nine times in ten it is one unticked checkbox |
| Understand the hardware | `NOTES.md` |
| Know why a rule is written the way it is | `NOTES.md` |

---

## The files

| File | What it is | Read it when |
|---|---|---|
| `README.md` | Setup and the button map | First. It is the only file you must read |
| `retror8-macmap.json` | The rule set. A bare `{description, manipulators}` object, seven manipulators | You are pasting it into Karabiner |
| `builder.html` | Assembles the rule set from dropdowns — [live here](https://charlieavocado.github.io/retror8-macmap/builder.html). Opens preloaded with the configuration above; no dependencies, nothing leaves the page | You want a different layout and would rather not hand-edit JSON |
| `RECIPES.md` | Around fifty ready-to-paste actions, each in its own copy box, plus a worksheet covering every input the mouse can offer | You are editing by hand, or planning on paper |
| `NOTES.md` | The lab notebook: what the hardware sends, what it refuses to send, and the reasoning behind every rule | Before changing anything structural, and any time something surprises you |
| `Retro-R8-Mouse-manual.pdf` | 8BitDo's own booklet, kept here so the button numbering and the mode chords are to hand | You need the diagram, or the left- and right-hand mode chords |
| `LICENSE` | MIT | — |

---

## What the rule set contains

Seven manipulators, in two tiers. Every one is scoped with `device_if` to the mouse's own
vendor and product IDs.

| # | Rule | Tier |
|---|---|---|
| 1 | Button 5 + `Ctrl` → delete forward | assigned |
| 2 | Button 5 → delete (backspace) | assigned |
| 3 | Button 4 → back | assigned |
| 4 | Wheel press → `Return` | assigned |
| 5 | Button 5 + `Ctrl`, factory keystrokes → delete forward | factory |
| 6 | Button 5, factory keystrokes → delete (backspace) | factory |
| 7 | Button 4, factory keystrokes → back | factory |

The **assigned** tier matches `F16`–`F19`, which the buttons send once they have been set in
8BitDo's Ultimate Software. The **factory** tier matches the keystrokes an unconfigured
mouse sends, so the set still works on a mouse that has never met that app or has been
factory reset. The two tiers match different inputs and cannot conflict.

Buttons 6 and 7 are deliberately absent. They send nothing until assigned in the vendor app,
and `F18` and `F19` do nothing on macOS, so they stay inert until you give them a job.

---

## The three-minute version

1. Tick **Modify events** for every R8 row in `Karabiner-Elements → Devices`. Both of them —
   the mouse presents two interfaces and each has its own checkbox.
2. Remove any Simple Modifications targeting the mouse.
3. In 8BitDo's Ultimate Software, set Buttons 4 to 7 to `F16`, `F17`, `F18`, `F19`.
4. Paste `retror8-macmap.json` into `Karabiner-Elements → Complex Modifications`.

Step 3 is the one people skip, and skipping it leaves Buttons 6 and 7 dead forever.

---

## Where the builder lives

**[charlieavocado.github.io/retror8-macmap](https://charlieavocado.github.io/retror8-macmap/)**
— the project page, with the builder one click away. The link is also in this repository's
**About** panel, top right of the repository home.

**Clicking `builder.html` in the file list does not open it.** GitHub always renders HTML
as source code, never as a page, and there is no setting that changes that. The only routes
to a working copy are the link above, or downloading the file and opening it locally — it is
self-contained and needs no network.

The site is served by GitHub Pages from `main` at the repository root, so every push
redeploys it.
