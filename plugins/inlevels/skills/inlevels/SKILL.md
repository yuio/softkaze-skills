---
name: inlevels
description: Read and change the projects of InLevels, the Windows application of audio meters (levels, loudness, spectrum, tuning) composed on a panel and kept as .inlv files - make the view the user describes, add, change and delete its components and groups, in the window open, with undo. Use whenever the user mentions InLevels, .inlv files, or wants meters, a layout or a view of their audio channels made.
---

# InLevels (.inlv)

This skill is for InLevels 1.1.

InLevels is a Windows application that measures the inputs of an audio device: each **component** shows one measure of one channel (a level graph, a loudness meter, an equalizer, a tuner...), and the components are placed freely on a panel, alone or in **groups**, and kept as a project, a `.inlv` file. Each window of the application has one project.

You change the project of a window that is open, through commands of the application: the user sees the change at once, and undoes it with Ctrl+Z, as each command is one step of undo. Do not write a `.inlv` file that a window has open.

## The commands
`InLevels.exe` is a command of the version of the Microsoft Store. Each command writes a JSON and ends at once; from PowerShell, pipe it (`| Out-String`) so it waits for its output. Every answer has `ok`, and `error` with the reason when it is false; the process then ends with code 2.

- `InLevels.exe --state` — the windows open, each with `mappings`, `name` (its file, or `Untitled`, `Untitled 2`... while it has none), `project` (the path, empty without a file), `modified`, `readOnly`, `live`, `changesAllowed` and, when false, `changesRefused` with why, the device (`api`, `device`, `deviceOpen`, `channels`, `sampleRate`) and how many `components` and `groups` it has. `-project <path or name>` tells only of that one, with `open`.
- `InLevels.exe --project [-out <file>]` — the project of the window, whole, with what is not saved, in the format of its file, with the `id` of each component and group. With `-out` it goes to that file and the answer says so.
- `InLevels.exe --paste <file>` — components and groups added, as in the format below. They keep the place they say, or take a free one when they have none, and are left selected. The answer has the ids of the new `components` and `groups`.
- `InLevels.exe --apply <file>` — a list of operations, done in order: added, changed and deleted. When one can not be done none is, and the error says which and why.
- `InLevels.exe --replace <file>` — the components, groups, presets and templates replaced by the ones of a whole project file, as `--project` gives it. The device and the view stay.

What `--apply` does not change, such as the channel mapping, is changed in the whole project: when the window has its project saved, with `modified` false, edit its file and the window loads it again at once, as one undo step; else take it with `--project`, change it and give it back with `--replace` at once. Either way the rest stays as it was.

With several windows open, every command but `--state` needs `-project`, with the path of the project or its name, as `--state` gives them. With one window, it is that one.

Before changing anything, run `--state`: when `changesAllowed` is false, tell the user `changesRefused` and do not insist. The user decides in File > Settings > "Changes of AI agents to the project" whether you can change it never, outside the live mode (the default) or always; a read only project can not be changed whatever that says. Then run `--project` to see what there is, and the ids.

Its `version` is the one of the application. When it is newer than the one this skill is for, tell the user once that the skill is older than their application, and that opening the application once brings the skill up to date by itself, which you will use the next time you are started.

## The panel
Places are in units of the panel, 96 dpi pixels at zoom 100%. The grid of the panel is 18 units, the height of a caption: place components on it, on multiples of 18, so they fit each other and the snap. A component is a rectangle `left`, `top`, `right`, `bottom`; its caption, 18 units tall, is inside it at its top. Components of a group are drawn without gaps between them as the group shows them: put them side by side, edge to edge. A group has no place of its own: it is the box around its components, with its own caption of 18 units above them, so leave a free row of the grid over each group.

The channels of the device are numbered from 0 in the file and from 1 for the user ("Channel 1" is `0`). A **mapped channel** has a name and is mapped to one channel of the device, as a microphone, or to several in their places, measured as one (its loudness of all of them, as BS.1770 adds them by their place): it is the channel `1000 + its id`. The project keeps them in `mappings` (`composites` in projects of before), a list of `{ "id": 1, "name": "Main L/R", "channels": [16, 17] }`: `channels` has a channel of the device per place, in the order left, right, center, LFE, left surround, right surround, `-1` for a place without one, as `[0, 1, 2, -1, 4, 5]` for 5.1 without LFE; one channel alone is a mono mapping. `--state` lists them with their `channel` and whether they are `available` with the device. A component may have a channel of the device or a mapped one.

## Components
`type` is one of these, with the settings each uses (`settings` keys, below):

| type | Shows | Settings |
| --- | --- | --- |
| `wave` | The envelope of the samples over time | `seconds`, `references` (dB) |
| `dbs` | The level in dBFS over time | `seconds`, magnifier, `references` (dB) |
| `lufs` | The loudness over time | `seconds`, magnifier, `references` (LUFS) |
| `geq` | 23 bands of a third of an octave, with peaks | magnifier, `peakHoldMs`, `peakFallDbs`, `references` (Hz) |
| `tuner` | The note, the cents and the frequency | `style`, `pitchWindow`, `references` (notes) |
| `meter` | The level as it reads: a classic VU, or the peaks in dBFS (PPM, sample peak, true peak); `vu` in projects of before | `meterMode`, `style`, `peakHoldMs`, `peakFallDbs`; for the peaks, magnifier and `references` (dBFS) |
| `lufsmeter` | The momentary loudness as a bar or a needle | `style`, magnifier, `references` (LUFS) |
| `pitch` | The heat of every semitone over time and the line of the note | `seconds`, `pitchTrace`, `pitchSpectrum`, `pitchLowOctave`, `pitchHighOctave`, `pitchWindow`, `references` (notes) |
| `info` | Values as text: LUFS, max LUFS, dB, max dB, Hz, note, and the loudness of EBU R128 since the last reset, the time and two alerts | `info`, `peakAlert`, `loudAlert`, `pitchWindow` |
| `correlation` | Of a mapped channel of two or more: how in phase its first two channels are, -1 to +1 | |
| `balance` | Of a mapped channel of two or more: how much louder its second channel is, in dB | |
| `vectorscope` | Of a mapped channel of two or more: its first two channels as a line through their samples of the last 60 ms, the mid up and the side across, with its scale automatic | |
| `log` | What the capture reports; one per project | |
| `notes` | Text with styles: `**bold**`, `*italic*`, `<u>underline</u>`, `~~struck~~`, emojis | `textSize`, `textColor`, `backColor` |
| `title` | A big centered text, all caption | `textSize`, `textColor` |
| `placeholder` | Nothing: it only takes a place | |

The sizes the application gives them, in units, caption included: graphs (`wave`, `dbs`, `lufs`, `geq`, `pitch`) 324 x 144; `tuner`, `meter` and `lufsmeter` 252 x 90 (a vertical one, taller than wide); `info` 144 x 144; `notes` 252 x 144; `log` 432 x 162; `title` 432 x 54. The smallest a component can be is 72 x 72.

A component in the file:

```json
{
    "id": 7,
    "type": "lufs",
    "channel": 0,
    "name": "Vocal",
    "left": 0, "top": 18, "right": 324, "bottom": 198,
    "group": 0,
    "preset": "",
    "text": "",
    "z": 0,
    "settings": { "seconds": 8, "color": 3, "captionSize": 1 }
}
```

- `name` — empty for the one of its type and channel ("LUFS · Ch 1").
- `group` — the index of its group in the `groups` list of the same file or of the same addition, -1 for none. In a change it is `groupId`, the id of the group, 0 for none.
- `preset` — the name of a preset of its type it follows, empty for none: it takes its settings, and a change of its settings changes the preset, and so every component of it.
- `text` — of `notes` and `title`.
- `z` — the order of painting: higher is in front, among the groups and the components of no group, or among the components of the same group. 0 to leave it in front when it is added.

### settings
All are optional: what is not given takes its default, or in a change stays as it is.

- `seconds` — of the time graphs: 1, 2, 4, 8, 16 or 32.
- `color` — of the caption: 0 is the one of the theme, 1 to 24 are Blue, Green, Orange, Purple, Teal, Red, Olive, Magenta, Sky, Lime, Amber, Violet, Cyan, Coral, Moss, Pink, Navy, Forest, Brown, Indigo, Slate, Maroon, Khaki, Grey.
- `captionSize` — 1 to 3 steps of the grid; `noCaptionGrouped` — true for no caption while it is in a group.
- `style` — of `tuner`, `meter` and `lufsmeter`: 0 horizontal, 1 vertical, 2 needle.
- `meterMode` — of `meter`: `"vu"` (its own scale, the magnifier and the references not used), `"ppm"`, `"sample"` or `"truepeak"` (dBFS, on a level axis from -60 to +3, red over 0, with the magnifier, which can go up to +3, and the references).
- `radar` — of `dbs` and `lufs`: true to draw them as a radar, the time going round once in `seconds` and the level away from the middle, as the loudness radars; best square.
- `radarArc` — of `dbs` and `lufs` as a radar: true for an arc around it lit up to the last level, as the loudness radars.
- Magnifier of the level axis of `dbs`, `lufs`, `geq` and `lufsmeter`: `lensFrom` and `lensTo`, the levels it magnifies (-60 to 0), and `lensHeight`, the percent of the height they take (10 to 95), 0 for none. The ones the user knows: Mix -30 to -12, Premaster -24 to -6, Master -12 to 0 for levels; Broadcast -29 to -17, Streaming -20 to -8, Premaster -24 to -12, Master -16 to -6 for loudness; all at 70.
- `peakHoldMs` (0 to 10000) and `peakFallDbs` (1 to 200) — of the peaks of the `geq`, and of the held peak of the `meter`.
- `pitchTrace` and `pitchSpectrum` — what the `pitch` shows; `pitchLowOctave` and `pitchHighOctave` — its note axis, from the C of one octave to the C of another, 1 to 8.
- `pitchWindow` — how the note is found: 2048 (slow, steady), 1024, 512 (the quickest, the default).
- `info` — the lines of the `info`: `{ "lufs": true, "lufsMax": true, "db": true, "dbMax": true, "hz": false, "note": false, "shortTerm": false, "integrated": false, "range": false, "truePeak": false, "plr": false, "sliding": false, "time": false, "peakAlert": false, "loudAlert": false }`. `range` is the loudness range (LRA), `truePeak` the highest true peak, `plr` the true peak over the integrated loudness, `sliding` the highest LUFS of the last 10 seconds, `time` the time since the reset; `peakAlert` and `loudAlert` light PEAK and LOUD. A wide info shows each line in one row, a narrow one in two.
- `peakAlert` (dBTP, -60 to 3) and `loudAlert` (LUFS, -60 to 0) — the limits of the alerts of the `info`: PEAK lights for 3 s when the true peak reaches it, LOUD while the short-term loudness is over it.
- `textSize` — of `notes` and `title`: 0 for the default, or 10, 12, 13, 15, 18, 22, 28, 36, 48, 64; `textColor` and `backColor` — as `color`.
- `references` — up to three lines of reference, a list of `{ "used": true, "name": "Target", "value": -14, "color": 2 }`, in the unit of the type (dB, LUFS, Hz, or a note as a MIDI number, 69 for A4). The graph above a reference takes its color; the highest wins.

## Groups
```json
{ "id": 3, "name": "Vocals", "color": 2, "captionSize": 1, "channel": -1, "template": "", "locked": false, "z": 0 }
```

- `channel` — the channel of all its components, -1 when each has its own.
- `locked` — the group stays where it is, and its content can not be moved, resized, added to or removed: to change a component of a locked group, unlock the group first, in the same list of operations, and lock it again after, if it was.
- `template` — the template it was made from; leave it as it is.

## The format of --paste
The one of the clipboard of InLevels:

```json
{
    "groups": [ { "name": "Vocals", "color": 2 } ],
    "meters": [
        { "type": "lufs", "channel": 0, "left": 0, "top": 18, "right": 324, "bottom": 198, "group": 0 },
        { "type": "tuner", "channel": 0, "left": 324, "top": 18, "right": 576, "bottom": 198, "group": 0 }
    ],
    "presets": { "lufs": [ { "name": "slow", "settings": { "seconds": 32 } } ] }
}
```

`presets` is optional: the ones the project does not have come with the components; the ones it has stay as they are.

## The format of --apply
```json
{
    "operations": [
        { "op": "add", "meters": [ { "type": "notes", "left": 0, "top": 216, "right": 288, "bottom": 396, "text": "**Mix notes**" } ] },
        { "op": "change", "group": 3, "locked": false },
        { "op": "change", "component": 7, "channel": 2, "settings": { "seconds": 16 } },
        { "op": "change", "component": 9, "groupId": 3, "left": 324, "top": 18, "right": 576, "bottom": 198 },
        { "op": "delete", "component": 12 },
        { "op": "delete", "group": 5 },
        { "op": "change", "group": 3, "locked": true }
    ]
}
```

- `add` — as a paste: `meters`, `groups`, `presets`.
- `change` of a `component` — any of its keys: `type`, `channel`, `name`, `left`, `top`, `right`, `bottom`, `groupId`, `preset`, `text`, `z`, `settings`. Of a `group` — `name`, `color`, `captionSize`, `channel`, `locked`, `z`.
- `delete` of a `component`, or of a `group` with all its components.

## Making a view the user describes
1. `--state`, to know the device: its channels and whether it is open, and whether you can change the project.
2. `--project -out <file>`, to see what there is.
3. Plan the places on the grid of 18 units, one group per channel or per instrument, the captions in mind; each component of the measure that answers what the user wants to see.
4. `--paste` or `--apply`, and tell the user what you made; they undo it with Ctrl+Z.

For a view from nothing, `--replace` with a whole project; or, to keep what there is, add next to it, to the right or below the components it has.

## Showing a project to the user
`InLevels.exe "<file>.inlv"` opens the project in a window, or brings to the front the one that has it.
