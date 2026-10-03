# Automation (Macros & Lua)

## Macros

Record a sequence of edits and movements and replay it.

- **Record** (++ctrl+shift+u++), **Play** (++ctrl+u++), and play N times.
- Save named macros, play them from a list, and bind them to F-keys.
- Macros are saved in a documented text format you can edit.

## Lua scripting

Run Lua scripts against the current document (**Script → Run Lua File**, or
**Run Last Script**). Scripts run in a sandbox (no file, OS or network access) and
every edit a script makes is a single **undoable** step.

Scripts work in text, large-file and hex tabs. Sample scripts ship with the app:
`sort_lines`, `remove_duplicate_lines`, `trim_trailing_whitespace`, `count_words`,
`list_todos`, `number_lines`, `uppercase_selection`, `fullwidth_to_ascii`,
`csv_to_markdown_table` and more.

### The `zume` API

Scripts talk to the document through a global `zume` table. Byte offsets are
0-based; a range is `[begin, end)`.

| Function | Description |
| --- | --- |
| `zume.text()` | The whole document as a string |
| `zume.size()` | Document size in bytes |
| `zume.caret()` | The caret's byte offset |
| `zume.set_caret(pos)` | Move the caret |
| `zume.selection()` | Two values: `begin, end` of the selection |
| `zume.select(begin, end)` | Set the selection |
| `zume.selected_text()` | The selected text |
| `zume.insert(pos, text)` | Insert text at an offset |
| `zume.replace(begin, end, text)` | Replace a byte range |
| `zume.line_count()` | Number of lines |
| `zume.line(i)` | Line `i` (0-based), without its line ending |
| `zume.read(offset, len)` | A bounded read (for large-file tabs) |
| `zume.log(msg)` | Write a line to the script log |
| `zume.api_version` | The API version (`2`) |

### Example scripts

**Uppercase the selection** (or the whole document when nothing is selected):

```lua
local b, e = zume.selection()
if e <= b then b, e = 0, zume.size() end
local text = zume.text():sub(b + 1, e)
zume.replace(b, e, text:upper())
zume.log(("uppercased %d bytes"):format(e - b))
```

**Sort the selected lines** (whole document when nothing is selected), as one
undo step:

```lua
local size = zume.size()
local b, e = zume.selection()
if e <= b then b, e = 0, size end
local text = zume.text()
while b > 0 and text:sub(b, b) ~= "\n" do b = b - 1 end   -- widen to line starts
while e < size and text:sub(e, e) ~= "\n" do e = e + 1 end
local block = text:sub(b + 1, e)
local trailing = block:sub(-1) == "\n"
if trailing then block = block:sub(1, -2) end
local lines = {}
for line in (block .. "\n"):gmatch("(.-)\r?\n") do lines[#lines + 1] = line end
table.sort(lines)
local out = table.concat(lines, "\n")
if trailing then out = out .. "\n" end
zume.replace(b, e, out)
zume.log(("sorted %d lines"):format(#lines))
```

**Remove duplicate lines**, keeping the first occurrence and the original order:

```lua
local text = zume.text()
local seen, out, removed = {}, {}, 0
local trailing = text:sub(-1) == "\n"
if trailing then text = text:sub(1, -2) end
for line in (text .. "\n"):gmatch("(.-)\r?\n") do
  if seen[line] then removed = removed + 1
  else seen[line] = true; out[#out + 1] = line end
end
local result = table.concat(out, "\n")
if trailing then result = result .. "\n" end
zume.replace(0, zume.size(), result)
zume.log(("removed %d duplicate line(s)"):format(removed))
```

**Number every line** (`"  12: text"`), using `line_count` / `line`:

```lua
local out, count = {}, zume.line_count()
for i = 0, count - 1 do
  out[#out + 1] = ("%4d: %s"):format(i + 1, zume.line(i))
end
zume.replace(0, zume.size(), table.concat(out, "\n"))
zume.log(("numbered %d lines"):format(count))
```

!!! tip
    An AI assistant can write these scripts for you. See
    [MCP → Let the AI write a Lua script](mcp.md#ai-lua).

## Project hooks

A project can run a script on open and before save (`on_open` / `on_save` in the
project settings), and keep project-local macros and scripts.
