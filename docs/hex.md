# Hex / Binary Editor

Switch any tab to a hex view with ++ctrl+shift+h++ (or the command palette).

- **Offset / hex / ASCII** columns, with the caret highlighted in both the hex and
  ASCII columns.
- **Select** bytes (Shift+movement or drag), **copy** them as hex or raw, and
  **find / replace** byte sequences (type `48 65` or `4865`, or plain text).
- **Insert / overwrite** bytes; edits share one undo history with the text view.
- Optional **EBCDIC** column, and a configurable **bytes-per-row** (8 / 16 / 32).
- Works on [large files](editing/large-files.md), and you can
  [compare two files byte-by-byte](compare.md).

!!! note "TODO"
    Add a screenshot here.
