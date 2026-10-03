# Search & Replace

## Find bar

- ++ctrl+f++ opens the find bar. ++f3++ / ++shift+f3++ go to the next / previous
  match; ++enter++ is next.
- Toggle options with ++alt+c++ (case), ++alt+r++ (regular expression) and
  ++alt+w++ (whole word).

## Find & Replace dialog

- ++ctrl+r++ opens the full dialog, with tabs for **Find**, **Replace**,
  **Find in Files** and **Replace in Files**.
- Multi-line search/replace fields (++ctrl+enter++ inserts a newline), capture
  groups (`$1`..`$9`) in replacements, and a search history (++up++ / ++down++).

## Scope

- Search the **current file**, the **selected text**, or **all open files**.
- With a column/block selection active, search is limited to those columns.

## Find in Files / Replace in Files

- Search a folder recursively with filename filters (e.g. `*.cpp, *.md`) and an
  include/exclude list.
- **Replace in Files** shows a **preview** of every change before you apply it,
  and the apply step is transactional (a failed write rolls everything back).

!!! note "TODO"
    Add a screenshot here.
