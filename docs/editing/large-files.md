# Large Files

Zume Editor opens very large files (**10&nbsp;GB and beyond**) without loading the
whole file into memory.

- **Instant open** - only the visible part is read; the first screen appears in
  about a second even on huge files.
- **Edit in place** - localized edits do not rewrite the whole file; saving is
  streamed and durable.
- **Streamed search** - literal and regular-expression search run through the file
  without freezing the UI, and wrap around.
- **Real line numbers** - a background index gives true line numbers and line/column
  on huge files.
- **Hex on large files** - the [hex editor](../hex.md) works on large files too.

Some features that need the whole file in memory (syntax highlighting, Git diff
gutters, folding) are off for very large files by design.
