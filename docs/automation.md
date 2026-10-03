# Automation (Macros & Lua)

## Macros

Record a sequence of edits and movements and replay it.

- **Record** (++ctrl+shift+u++), **Play** (++ctrl+u++), and play N times.
- Save named macros, play them from a list, and bind them to F-keys.
- Macros are saved in a documented text format you can edit.

## Lua scripting

Run Lua scripts against the current document (**Script → Run Lua File**).

- A sandboxed standard library plus a `zume` table (text, caret, selection, insert,
  replace, lines, log, ...). All edits are undoable.
- Scripts work in text, large-file and hex tabs.
- Sample scripts ship with the app (sort lines, remove duplicates, CSV to Markdown
  table, and more).

## Project hooks

A project can run a script on open and before save (`on_open` / `on_save` in the
project settings), and keep project-local macros and scripts.

!!! note "TODO"
    Add a screenshot here.
