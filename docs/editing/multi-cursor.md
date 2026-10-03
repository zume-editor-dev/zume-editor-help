# Multi-cursor & Column mode

## Multiple cursors

- **Alt+Click** adds a cursor; ++ctrl+alt+up++ / ++ctrl+alt+down++ stack cursors
  above / below. ++esc++ collapses back to one.
- Typing, Backspace/Delete and movement apply to every cursor; overlapping
  cursors merge. A multi-cursor edit is a single undo step.

## Column / block mode

- **Shift+Alt+drag** selects a rectangle. Typing or deleting acts per line.
- **Toggle Column Mode** (++alt+shift+c++) makes Shift+movement build a block; in
  this mode a plain drag also builds a rectangle.

## Column tools

- **Column Fill** - insert the same text on every line of the block.
- **Column Number** - insert an incrementing number (`start[,step]`), zero-padded.
- **Align at Carets** / **Align on Delimiter** - line up columns.

!!! note "TODO"
    Add a screenshot here.
