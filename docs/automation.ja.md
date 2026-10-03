# 自動化（マクロ & Lua）

## マクロ

編集・移動の操作を記録して再生できます。

- **記録**（++ctrl+shift+u++）、**再生**（++ctrl+u++）、N回再生。
- 名前付きマクロの保存、一覧からの再生、Fキーへの割り当て。
- マクロは編集可能なテキスト形式で保存されます。

## Luaスクリプト

現在の文書に対してLuaスクリプトを実行できます（**Script → Run Lua File** / **Run Last
Script**）。スクリプトはサンドボックス（ファイル・OS・ネットワークへのアクセス不可）で
実行され、スクリプトによる編集は1回で取り消せる**1ステップ**になります。

テキスト / 大容量 / Hex のいずれのタブでも動作します。サンプルスクリプトを同梱:
`sort_lines`、`remove_duplicate_lines`、`trim_trailing_whitespace`、`count_words`、
`list_todos`、`number_lines`、`uppercase_selection`、`fullwidth_to_ascii`、
`csv_to_markdown_table` など。

### `zume` API

スクリプトはグローバルの `zume` テーブルを通して文書を操作します。バイトオフセットは
0始まり、範囲は `[begin, end)` です。

| 関数 | 説明 |
| --- | --- |
| `zume.text()` | 文書全体を文字列で取得 |
| `zume.size()` | 文書のバイトサイズ |
| `zume.caret()` | キャレットのバイトオフセット |
| `zume.set_caret(pos)` | キャレットを移動 |
| `zume.selection()` | 選択範囲の `begin, end`（2値） |
| `zume.select(begin, end)` | 選択範囲を設定 |
| `zume.selected_text()` | 選択テキスト |
| `zume.insert(pos, text)` | 指定位置にテキスト挿入 |
| `zume.replace(begin, end, text)` | バイト範囲を置換 |
| `zume.line_count()` | 行数 |
| `zume.line(i)` | i 行目（0始まり、改行なし） |
| `zume.read(offset, len)` | 範囲限定の読み取り（大容量タブ用） |
| `zume.log(msg)` | スクリプトログに1行出力 |
| `zume.api_version` | API バージョン（`2`） |

### スクリプト例

**選択範囲を大文字化**（未選択なら文書全体）:

```lua
local b, e = zume.selection()
if e <= b then b, e = 0, zume.size() end
local text = zume.text():sub(b + 1, e)
zume.replace(b, e, text:upper())
zume.log(("uppercased %d bytes"):format(e - b))
```

**選択行をソート**（未選択なら全行）。1回の取り消しで戻せます:

```lua
local size = zume.size()
local b, e = zume.selection()
if e <= b then b, e = 0, size end
local text = zume.text()
while b > 0 and text:sub(b, b) ~= "\n" do b = b - 1 end   -- 行頭まで広げる
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

**重複行の削除**（最初の出現と元の順序を保持）:

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

**各行に行番号を付ける**（`"  12: text"`）。`line_count` / `line` を使用:

```lua
local out, count = {}, zume.line_count()
for i = 0, count - 1 do
  out[#out + 1] = ("%4d: %s"):format(i + 1, zume.line(i))
end
zume.replace(0, zume.size(), table.concat(out, "\n"))
zume.log(("numbered %d lines"):format(count))
```

!!! tip
    これらのスクリプトは AI に書かせることもできます。
    [MCP → AI に Lua スクリプトを書かせる](mcp.md#ai-lua) を参照してください。

## プロジェクトフック

プロジェクトは、開いたときと保存前にスクリプトを実行できます（プロジェクト設定の
`on_open` / `on_save`）。プロジェクト専用のマクロやスクリプトも保持できます。
