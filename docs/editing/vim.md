# Vim mode

Zume Editor has an optional **Vim-style modal editing** mode for text tabs. It
is off by default and does not need any plugin.

## Turning it on

- **Edit → Toggle Vim Mode**, or type `:set vim` (and `:set novim` to turn it
  off) once Vim mode is on.
- The setting is **remembered** and applies to **every text tab**.
- Vim mode works on text tabs only. Hex, large-file and Git tabs ignore it.

The status bar shows the current mode: `-- NORMAL --`, `-- INSERT --`,
`-- VISUAL --`, `-- V-LINE --` or `-- V-BLOCK --`.

!!! note "Ctrl shortcuts still work"
    The usual shortcuts (++ctrl+s++ save, ++ctrl+f++ find, ++ctrl+shift+p++
    command palette, …) keep working in every mode. Two NORMAL-mode keys follow
    Vim instead: ++ctrl+r++ is **redo** and ++ctrl+v++ enters **visual-block**.

## Modes

| Enter | Mode |
| --- | --- |
| `i` `a` `I` `A` `o` `O` | INSERT (type text; ++esc++ returns to NORMAL) |
| `v` | VISUAL (charwise) |
| `V` | VISUAL LINE |
| ++ctrl+v++ | VISUAL BLOCK |
| `:` `/` `?` | command line (ex / search) |

## Motions

`h` `j` `k` `l` (and the arrow keys), `w` `b` `e`, `0` `^` `$`, `gg` `G`,
`{count}G` (go to line), and `f` `t` `F` `T` to a character with `;` / `,` to
repeat. Counts work: `3j`, `42G`, `2w`.

## Operators and text objects

- Operators: `d` (delete), `c` (change), `y` (yank), `>` / `<` (indent /
  dedent), `gu` / `gU` (lower / upper case).
- Combine an operator with a motion (`dw`, `c$`, `y3j`) or **double** it for
  whole lines (`dd`, `yy`, `cc`, `>>`).
- Text objects: `iw` / `aw`, and `i` / `a` with `"` `'` `` ` `` `(` `{` `[`
  `<` — e.g. `ci"` changes inside quotes, `da(` deletes around parentheses.

## Actions

`x` `D` `C` `p` `P` `J` `~` `r{char}`, undo `u` / redo ++ctrl+r++, and `.` to
repeat the last change. Named registers with a leading `"x` (e.g. `"ayy` then
`"ap`); the unnamed register backs plain `y` / `d` / `p`.

## Search and replace

- `/pattern` and `?pattern` search forward / backward; `n` / `N` repeat, and
  `*` / `#` search the word under the cursor. The command line is shown in the
  status bar.
- `:[range]s/pattern/replacement/[g][i]` substitutes (`:%s/…/…/g` for the whole
  file, `:s/…/…/` for the current line); `:noh` clears the match highlight.

## Ex commands

`:w [name]` save, `:q` close the tab (`:q!` to discard, `:wq` / `:x` save and
close), `:qa` / `:wqa` for all tabs, `:{number}` go to a line, and
`:set vim` / `:set novim`.

!!! note
    Vim mode and multi-cursor / column mode are not used at the same time.
