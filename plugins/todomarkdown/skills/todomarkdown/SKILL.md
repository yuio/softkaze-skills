---
name: todomarkdown
description: Read and edit TodoMarkdown to do lists (.tdmd files). Use whenever a .tdmd file is read, created or modified, or the user mentions TodoMarkdown.
---

# TodoMarkdown (.tdmd)

A plain text to do list, in UTF-8. Each line of the file is one item; never wrap an item over several lines.

## Lines
- `### Title` — section, never indented. It owns everything up to the next section. Sections are never nested: `## Title` and `# Title` are sections too, with a bigger title, `#` being the biggest. Write `###` unless a bigger one is asked for.
- `- [ ] text` — pending task
- `- [x] text` — done task
- `- [40%] text` — partly done task (integer from 1 to 99)
- `- [-] text` — cancelled task
- `- text` — note: an item without a check
- `text` — any other line is an item of plain text; prefer a note
- `<!-- text -->` — comment, on a line of its own
- `<!-- title: text -->` — optional title of the document, among the first comments of the file. Without it, the title is the file name.

Items can come before the first section. Blank lines are kept and mean nothing.

## Nesting
An item is a child of the item above it when it is indented one level more. Use the indentation the file already has (four spaces per level by default) and never mix.

```
### Trip to Lisbon
- [ ] Find a place to stay
    - [x] Choose the neighbourhood
    - [ ] Confirm the booking
    - the one near the river has **no lift**
```

## Text
Inside the text of an item: `**bold**`, `*italic*`, `<u>underline</u>`, and `<br>` for a line break. No other markdown.

- A `*` that is text and not a mark is written `&#42;`, and the `<` of a `<u>` or `</u>` that is text is written `&lt;`. Leave the ones the file already has as they are.
- An address that starts with `http://` or `https://` is shown as a link; write it as it is, without markdown around it.
- A file is linked as `<file:E:/docs/notes.tdmd>`, or relative to the folder of the document as `<file:./notes.tdmd>` or `<file:../plans/next.tdmd>`; between `<` and `>` its path may have blanks. Link a file only when it exists and the user may want to open it, not in examples.
- Emojis are characters of the text as any other.

## Text size
A task or a note can have a bigger text: heading marks right before its text, after the check when it has one. `#` is the biggest of three sizes, `###` the smallest. The title of a section has the same three sizes, and none smaller.

```
- [ ] # The biggest
- [ ] ## Big
- ### A note a little bigger
```

## Colors
A task, a note or a section can have a color for its text and another for what is behind it: an emoji right before the text, after the marks of its size when it has them. A circle is the color of the text and a square the one behind it, in red, orange, yellow, green, blue or purple: 🔴 🟠 🟡 🟢 🔵 🟣 and 🟥 🟧 🟨 🟩 🟦 🟪. One of each at most; write the square first.

```
- [ ] 🔴 Text in red
- [ ] 🟨🔴 Text in red over yellow
- [ ] ## 🟦 Big, over blue
### 🟩 A section with a green header
```

Any other emoji, or one of those anywhere else in the text, is text.

## Progress
- The progress of a task with tasks inside is always computed from them, and its own mark is ignored. To change it, edit its children; write `[ ]` on a new one.
- Keep the mark of such a task in step with them, as other markdown viewers show what is written: `[x]` when all its tasks are done, `[ ]` otherwise.
- A cancelled task is out of the progress, with everything inside it.

## Rules when editing
- Keep every line you were not asked to change exactly as it is.
- Ignore the `.tdmd` folder next to a document: it is state of the application, of each user. In a git repository it is best left out of it: when you work with `.tdmd` files in one whose `.gitignore` does not have `.tdmd/`, suggest adding it.

## Before changing a file
The user may have the file open in the application with changes not saved, or with an item being edited: writing it then makes them choose between their changes and yours. On Windows, before changing a `.tdmd` file, ask the application:

```
TodoMarkdown.exe --state "<path>"
```

It writes a JSON and ends at once. `open` tells whether a window has the file, and each one of `windows` has `modified` (changes not saved), `editing` (the text of an item is being edited, and is not in the document yet), `readOnly` and `encrypted`. When `modified` or `editing` is true, do not write the file: ask the user to save it, or to finish the edit, and ask the application again. Without a path, it tells the state of all the windows open.

- `TodoMarkdown.exe` is a command of the version of the Microsoft Store. Otherwise, run it with its full path, which is in the command that opens the `.tdmd` files: `reg query "HKCU\Software\Classes\TodoMarkdown.tdmd\shell\open\command"`. When it is not found, or it does not know `--state`, go on without it.
- From PowerShell, pipe it (`| Out-String`) so it waits for its output.

## Encrypted documents
A file whose first line starts with `<!-- TodoMarkdown encrypted document.` is encrypted with a password; the rest of it is base64. Do not modify it or try to read it: tell the user it has a password, which is removed in the application with File > Remove Password.

## Showing a file to the user
`.tdmd` files open in the TodoMarkdown application when it is installed. When the user asks to see one, run `start "" "<path>"` on Windows or `open "<path>"` on macOS.
