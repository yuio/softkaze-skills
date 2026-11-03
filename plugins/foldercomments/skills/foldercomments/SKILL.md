---
name: foldercomments
description: Read and write the comments FolderComments keeps for folders (notes, a fast comment and a color) in the hidden .foldercomments folder inside each folder. Use whenever a .foldercomments folder is found, or the user mentions FolderComments or the comments, notes or colors of their folders.
---

# FolderComments (.foldercomments)

This skill is for FolderComments 1.0.

FolderComments is a Windows application that gives each folder notes of any length, a fast comment of one line and a color, shown in a tree of the folder. Only folders have them, not files.

## Where they are
What FolderComments keeps of a folder is inside it, in a hidden folder called `.foldercomments`:

- `notes.md` — the notes, in UTF-8.
- `info.json` — the fast comment, the color and the filter of the tree.
- `notes.md.recovery` — a copy of notes not saved yet, written by the application. Do not change it.

Nothing is kept anywhere else: those hidden folders are all there is of the comments.

- The folder is only there while it has something. When you make it, make it hidden: `attrib +h "<folder>\.foldercomments"`. When you leave it empty, remove it.
- It is not junk, a cache nor an output of a build: keep it when copying, moving or cleaning folders, and never delete one the user did not ask to delete.
- When you only work with the files of a folder, leave its `.foldercomments` as it is.

## notes.md
Markdown with a few of its marks only, the same as the notes of TodoMarkdown:

- A line can be a heading: `# ` is the biggest of three sizes and `### ` the smallest.
- A line can have a color for its text and another for what is behind it: an emoji at its start, after the heading marks when it has them. A circle is the color of the text and a square the one behind it, in red, orange, yellow, green, blue or purple: 🔴 🟠 🟡 🟢 🔵 🟣 and 🟥 🟧 🟨 🟩 🟦 🟪. One of each at most, the square first, and a blank after them.
- Inside the text: `**bold**`, `*italic*`, `<u>underline</u>` and `~~strikethrough~~`.
- Nothing else is markdown: lists, links, tables and code are plain text, shown as they are written. An address that starts with `http://` or `https://` opens with Ctrl and a click; write it as it is.
- A `*` that is text and not a mark is written `&#42;`, a `~` `&#126;`, the `<` of a `<u>` or `</u>` that is text `&lt;`, and a `\` goes before the `#` that start a line of text that is not a heading. Leave the ones the file already has as they are.
- Any other emoji, or one of those anywhere else in the line, is text.

```
# 🟨 Low Tide Radio
**Tempo:** 92 BPM   **Key:** D minor
## Structure
🟠 Chorus 1  0:52 - 1:24   vocal *1.5 dB* too loud
~~Comp the lead vocal~~
```

Keep every line you were not asked to change exactly as it is, with the ends of line the file has.

## info.json
```
{
    "fastComment": "Mixing v3 · vocal too loud in the choruses",
    "color": "#F4D35E"
}
```

- `fastComment` — one line, shown after the name of the folder in the tree.
- `color` — `#RRGGBB`, painted behind the name of the folder in the tree; its text is black or white as the color asks. Soft colors read best.
- `filter` — what the tree shows when that folder is opened in the application: `commentedOnly`, `currentLevelOnly`, `hideFiles`, `showHidden`, `hideDotFiles` and `hideDotFolders`, each `true` or `false`. Leave it as it is unless the user asks.

All of them are optional, and no other key is kept. When none is left, delete the file.

## Locked comments
The user locks the comments of a folder in the application to make them read only: `notes.md` and `info.json` have the read only attribute then. Do not change them, nor take that attribute away: tell the user they are locked, which they undo with the lock of the application.

## Before changing comments
The user may have the comments of the folder open in the application with changes not saved: writing them then makes the user choose between their changes and yours. Before changing the comments of a folder, ask the application:

```
FolderComments.exe --state "<folder>"
```

It writes a JSON and ends at once. `open` tells whether a window has the comments of that folder in its editor, and each one of `windows` has `folder` (the folder the window shows), `selected` (the folder whose comments are in its editor), `modified` (its notes or its fast comment have changes not saved) and `readOnly` (they are locked). When `modified` is true, do not write them: ask the user to save them (Ctrl+S, or Enter in the fast comment), and ask the application again. Without a folder, it tells the state of all the windows open.

- `FolderComments.exe` is a command of the version of the Microsoft Store. When it is not found, or it does not know `--state`, go on without it.
- From PowerShell, pipe it (`| Out-String`) so it waits for its output.
- Its `version` is the one of the application. When it is newer than the one this skill is for, tell the user once that the skill is older than their application, and that opening the application once brings the skill up to date by itself, which you will use the next time you are started.

## After changing comments
The window that has the folder in its editor reads the changes by itself when it is active, asking the user first when the settings say so. The fast comments and colors of the other folders of its tree are shown again with Refresh (F5).

## Showing a folder to the user
`FolderComments.exe "<folder>"` opens the folder in a window of the application, or brings to the front the one that has it.
