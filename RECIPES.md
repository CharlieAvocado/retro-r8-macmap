# Recipes — actions to paste into `retro-r8-macmap.json`

Every block below is a complete `to` line. Find the manipulator you want to change by its
`description`, delete its existing `to` line, and paste one of these in its place.

For an interactive version that assembles the whole file for you, open `builder.html` in a
browser. It needs no installation and sends nothing anywhere.

**If you change Button 4 or Button 5, change its `FACTORY FIRMWARE` twin too.** Those
manipulators exist so the rule set still works on a mouse that has never met the vendor app,
or has been factory reset. Editing only one half gives a mouse that behaves differently
depending on which firmware state it is in.

---

## Worksheet

Every input the mouse can offer, and room to record what you want from it. Triggers assume
the `F16`–`F19` assignment from the README. Nothing here is mandatory — most people fill in
a handful of rows and leave the rest empty.

Each button carries two rows because a click and a hold are separate actions. Note the
trade: once a button has a hold, its click fires on release rather than on press.
`builder.html` generates clicks, holds, modifier variants and chords — everything in this
worksheet except a chord involving the wheel press.

| Input | Trigger | Your choice |
|---|---|---|
| Button 4, clicked | `f16` | |
| Button 4, held | `f16` held | |
| Button 4 + `Ctrl` | `f16` + `control` | |
| Button 4 + `Opt` | `f16` + `option` | |
| Button 4 + `Cmd` | `f16` + `command` | |
| Button 4 + `Shift` | `f16` + `shift` | |
| Button 5, clicked | `f17` | |
| Button 5, held | `f17` held | |
| Button 5 + `Ctrl` | `f17` + `control` | |
| Button 5 + `Opt` | `f17` + `option` | |
| Button 5 + `Cmd` | `f17` + `command` | |
| Button 5 + `Shift` | `f17` + `shift` | |
| Button 6, clicked | `f18` | |
| Button 6, held | `f18` held | |
| Button 6 + `Ctrl` | `f18` + `control` | |
| Button 6 + `Opt` | `f18` + `option` | |
| Button 6 + `Cmd` | `f18` + `command` | |
| Button 6 + `Shift` | `f18` + `shift` | |
| Button 7, clicked | `f19` | |
| Button 7, held | `f19` held | |
| Button 7 + `Ctrl` | `f19` + `control` | |
| Button 7 + `Opt` | `f19` + `option` | |
| Button 7 + `Cmd` | `f19` + `command` | |
| Button 7 + `Shift` | `f19` + `shift` | |
| Wheel press, clicked | `button3` | |
| Wheel press, held | `button3` held | |
| Buttons 4 + 5 together | `simultaneous` | |
| Buttons 4 + 6 together | `simultaneous` | |
| Buttons 4 + 7 together | `simultaneous` | |
| Buttons 5 + 6 together | `simultaneous` | |
| Buttons 5 + 7 together | `simultaneous` | |
| Buttons 6 + 7 together | `simultaneous` | |

Scroll direction is deliberately absent. Karabiner cannot trigger on it; `NOTES.md`
explains what the wheel can do instead.

---

## Navigation

### Back
```json
"to": [{ "key_code": "open_bracket", "modifiers": ["left_command"] }]
```

### Forward
```json
"to": [{ "key_code": "close_bracket", "modifiers": ["left_command"] }]
```

### Page up
```json
"to": [{ "key_code": "page_up" }]
```

### Page down
```json
"to": [{ "key_code": "page_down" }]
```

### Top of document
```json
"to": [{ "key_code": "up_arrow", "modifiers": ["left_command"] }]
```

### Bottom of document
```json
"to": [{ "key_code": "down_arrow", "modifiers": ["left_command"] }]
```

---

## Editing

### Delete (backspace)
```json
"to": [{ "key_code": "delete_or_backspace" }]
```

### Delete forward
```json
"to": [{ "key_code": "delete_forward" }]
```

### Copy
```json
"to": [{ "key_code": "c", "modifiers": ["left_command"] }]
```

### Cut
```json
"to": [{ "key_code": "x", "modifiers": ["left_command"] }]
```

### Paste
```json
"to": [{ "key_code": "v", "modifiers": ["left_command"] }]
```

### Paste without formatting
```json
"to": [{ "key_code": "v", "modifiers": ["left_command", "left_option", "left_shift"] }]
```

### Undo
```json
"to": [{ "key_code": "z", "modifiers": ["left_command"] }]
```

### Redo
```json
"to": [{ "key_code": "z", "modifiers": ["left_command", "left_shift"] }]
```

### Select all
```json
"to": [{ "key_code": "a", "modifiers": ["left_command"] }]
```

### Find
```json
"to": [{ "key_code": "f", "modifiers": ["left_command"] }]
```

### Save
```json
"to": [{ "key_code": "s", "modifiers": ["left_command"] }]
```

### Return
```json
"to": [{ "key_code": "return_or_enter" }]
```

### Escape
```json
"to": [{ "key_code": "escape" }]
```

---

## Tabs and windows

### Previous tab
```json
"to": [{ "key_code": "tab", "modifiers": ["left_control", "left_shift"] }]
```

### Next tab
```json
"to": [{ "key_code": "tab", "modifiers": ["left_control"] }]
```

### New tab
```json
"to": [{ "key_code": "t", "modifiers": ["left_command"] }]
```

### Reopen closed tab
```json
"to": [{ "key_code": "t", "modifiers": ["left_command", "left_shift"] }]
```

### Close window or tab
```json
"to": [{ "key_code": "w", "modifiers": ["left_command"] }]
```

### Switch application
```json
"to": [{ "key_code": "tab", "modifiers": ["left_command"] }]
```

---

## Spaces and Mission Control

### Mission Control
```json
"to": [{ "key_code": "mission_control" }]
```

### Application windows
```json
"to": [{ "key_code": "down_arrow", "modifiers": ["left_control"] }]
```

### Previous desktop
```json
"to": [{ "key_code": "left_arrow", "modifiers": ["left_control"] }]
```

### Next desktop
```json
"to": [{ "key_code": "right_arrow", "modifiers": ["left_control"] }]
```

### Launchpad
```json
"to": [{ "key_code": "launchpad" }]
```

---

## System

### Spotlight
```json
"to": [{ "key_code": "spacebar", "modifiers": ["left_command"] }]
```

### Screenshot of an area
```json
"to": [{ "key_code": "4", "modifiers": ["left_command", "left_shift"] }]
```

### Screenshot of the whole screen
```json
"to": [{ "key_code": "3", "modifiers": ["left_command", "left_shift"] }]
```

### Lock screen
```json
"to": [{ "key_code": "q", "modifiers": ["left_control", "left_command"] }]
```

### Zoom in
```json
"to": [{ "key_code": "equal_sign", "modifiers": ["left_command"] }]
```

### Zoom out
```json
"to": [{ "key_code": "hyphen", "modifiers": ["left_command"] }]
```

---

## Media

### Volume up
```json
"to": [{ "consumer_key_code": "volume_increment" }]
```

### Volume down
```json
"to": [{ "consumer_key_code": "volume_decrement" }]
```

### Mute
```json
"to": [{ "consumer_key_code": "mute" }]
```

### Play or pause
```json
"to": [{ "consumer_key_code": "play_or_pause" }]
```

### Next track
```json
"to": [{ "consumer_key_code": "scan_next_track" }]
```

### Previous track
```json
"to": [{ "consumer_key_code": "scan_previous_track" }]
```

### Screen brighter
```json
"to": [{ "key_code": "display_brightness_increment" }]
```

### Screen dimmer
```json
"to": [{ "key_code": "display_brightness_decrement" }]
```

---

## Beyond single keystrokes

### Do nothing — disable the button
```json
"to": [{ "key_code": "vk_none" }]
```

### Open an application
```json
"to": [{ "shell_command": "open -a 'Logic Pro'" }]
```

### Run a shell command
```json
"to": [{ "shell_command": "osascript -e 'display notification \"hello\"'" }]
```

### Send several keystrokes in order
```json
"to": [
  { "key_code": "c", "modifiers": ["left_command"] },
  { "key_code": "tab", "modifiers": ["left_command"] },
  { "key_code": "v", "modifiers": ["left_command"] }
]
```

### Different action on tap and on hold

This one replaces the whole manipulator rather than just its `to` line, because a tap and a
hold are two separate keys in the JSON.

```json
{
  "description": "B6 (F18) -> Mission Control on tap, Launchpad on hold",
  "from": {
    "key_code": "f18",
    "modifiers": { "optional": ["caps_lock"] }
  },
  "to_if_alone": [{ "key_code": "mission_control" }],
  "to_if_held_down": [{ "key_code": "launchpad" }],
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

### One action in a named application only

Add this alongside the existing `device_if` condition, inside the same `conditions` array.

```json
{
  "type": "frontmost_application_if",
  "bundle_identifiers": ["^com\\.apple\\.logic10$"]
}
```
