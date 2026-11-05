# 自动化（宏与 Lua）

## 宏

录制一系列编辑与移动操作并回放。

- **录制**（++ctrl+shift+u++）、**播放**（++ctrl+u++），以及播放 N 次。
- 保存命名宏、从列表中播放，并将其绑定到 F 键。
- 宏以一种有文档说明的文本格式保存，你可以直接编辑。

## Lua 脚本

针对当前文档运行 Lua 脚本（**Script → Run Lua File**，或
**Run Last Script**）。脚本在沙箱中运行（无法访问文件、操作系统或网络），且脚本所做的每一次编辑都是
一个可**撤销**的步骤。

脚本可在文本、大文件和十六进制标签页中工作。应用随附了示例脚本：
`sort_lines`、`remove_duplicate_lines`、`trim_trailing_whitespace`、`count_words`、
`list_todos`、`number_lines`、`uppercase_selection`、`fullwidth_to_ascii`、
`csv_to_markdown_table` 等。

### `zume` API

脚本通过全局 `zume` 表与文档交互。字节偏移从 0 开始；区间为 `[begin, end)`。

| 函数 | 说明 |
| --- | --- |
| `zume.text()` | 整篇文档（字符串） |
| `zume.size()` | 文档的字节大小 |
| `zume.caret()` | 光标的字节偏移 |
| `zume.set_caret(pos)` | 移动光标 |
| `zume.selection()` | 两个值：选区的 `begin, end` |
| `zume.select(begin, end)` | 设置选区 |
| `zume.selected_text()` | 选中的文本 |
| `zume.insert(pos, text)` | 在某偏移处插入文本 |
| `zume.replace(begin, end, text)` | 替换一段字节区间 |
| `zume.line_count()` | 行数 |
| `zume.line(i)` | 第 `i` 行（从 0 开始），不含行结束符 |
| `zume.read(offset, len)` | 有界读取（用于大文件标签页） |
| `zume.log(msg)` | 向脚本日志写入一行 |
| `zume.api_version` | API 版本（`2`） |

### 示例脚本

**将选区转为大写**（未选中任何内容时则转换整篇文档）：

```lua
local b, e = zume.selection()
if e <= b then b, e = 0, zume.size() end
local text = zume.text():sub(b + 1, e)
zume.replace(b, e, text:upper())
zume.log(("uppercased %d bytes"):format(e - b))
```

**对选中的行排序**（未选中任何内容时则处理整篇文档），作为单次撤销步骤：

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

**删除重复行**，保留首次出现的行及原始顺序：

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

**为每一行编号**（`"  12: text"`），使用 `line_count` / `line`：

```lua
local out, count = {}, zume.line_count()
for i = 0, count - 1 do
  out[#out + 1] = ("%4d: %s"):format(i + 1, zume.line(i))
end
zume.replace(0, zume.size(), table.concat(out, "\n"))
zume.log(("numbered %d lines"):format(count))
```

!!! tip
    AI 助手可以为你编写这些脚本。参见
    [MCP → 让 AI 编写 Lua 脚本](mcp.md#ai-lua)。

## 项目钩子

项目可以在打开时和保存前运行脚本（项目设置中的 `on_open` / `on_save`），
并保留项目本地的宏与脚本。
