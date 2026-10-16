# セキュリティ解析

Zume Editor には、バイナリ診断・データ変換・シグネチャ走査のツールがあり、
リバースエンジニアリング、CTF、認可されたセキュリティ作業に役立ちます。いずれも
[MCP サーバー](mcp.md#tools)経由で AI エージェントからヘッドレスに利用できます
（`entropy`・`transform`・`yara_scan`・`pe_info` ツール）。

## PE 情報

**コード → PE 情報…** は Windows の EXE / DLL を構造として読み取り、結果パネルに
一覧します：ヘッダーとセキュリティフラグ（ASLR / DEP / CFG）、各セクションとその
エントロピー、インポートする DLL と関数、エクスポート、そして機能の目録を作るのに
必要なリソース — バージョン文字列、文字列テーブル、各ダイアログのキャプションと
コントロール、メニュー項目、マニフェスト — に加えて PDB パスと最終セクション以降に
付加されたデータです。

アクティブなタブ自体が PE ファイルなら（16 進ビューで開いたバイナリなど）その
ファイルを読み、そうでなければファイル選択ダイアログで指定します。読み込みは
ワーカースレッドで行うので、大きなバイナリでもエディターは止まりません。行を
クリック（または **F4**）するとその行をクリップボードにコピーし、**Esc** で
パネルを閉じます。

コードを指す行はコピーではなく逆アセンブル表示へジャンプします：エントリポイントの
行、コードアドレスを持つエクスポート、CODE / EXECUTE 属性のセクションです。
クリックするとそのファイルを 16 進ビューで該当オフセットに開き、ファイルが宣言する
アーキテクチャ（x64 / x86 / ARM64 / ARM / Thumb）で **16進 → 逆アセンブル** を
実行するので、結果パネルがそのオフセットからの逆アセンブルに切り替わります。
16 進ビューはそのまま残るので、続けて参照できます。

![結果パネルの PE 情報](assets/screenshots/pe_info_panel.png){ loading=lazy }

同じ読み取りエンジンが MCP ツール `pe_info` です（[MCP](mcp.md#tools) 参照）。
逆アセンブル・ロード・実行は一切行いません。解析はライセンスで許されたバイナリに
限ってください。

## YARA スキャン

**コード → YARA スキャン…** は、YARA ルールでアクティブ文書を走査します。

ダイアログに入力するのは、**YARA ルールファイルのパス**（`.yar` / `.yara`）です
— ルール本文そのものではありません。先にルールをファイルに書いて保存し、その場所を
ダイアログで指定します。

1. ルールファイルを作成して保存（下の例を参照）。例：`C:\rules\malware.yar`、
   `~/rules/strings.yara`。
2. 走査したい文書を開き、**コード → YARA スキャン…** を実行（コマンドパレットにも
   あります）。
3. ルールファイルのフルパスを入力して Enter。
4. 一致は結果パネルに `rule  $string  line N @ offset` の形で一覧表示されます。
   行をクリック（または F4）でその行へジャンプします。

対象は通常のテキスト / コードタブです（大容量・HEX・Git タブでは無効）。

### ルールファイルの例

```yara
rule suspicious_strings
{
    strings:
        $a  = "cmd.exe /c" nocase
        $b  = "powershell" nocase wide
        $mz = { 4D 5A ?? 00 }              // PE ヘッダ（?? は任意バイト）
        $url = /https?:\/\/[a-z0-9.\-]+/ nocase
    condition:
        $mz and (2 of ($a, $b, $url))
}
```

### 対応している記法

YARA の実用的なサブセット：

- **文字列** — テキスト（`"..."`）、hex（`{ 4D 5A ?? 00 }`、`??`=任意バイト、
  `?`=ニブルのワイルドカード）、正規表現（`/regex/`）。
- **文字列修飾子** — `nocase` / `wide` / `ascii` / `fullword`。
- **条件** — 文字列名（`$a`）、`all of them` / `any of them` / `N of them`、
  `N of ($a*)`、`#a <op> N`（出現回数）、`$a at N`（オフセット一致）、
  `filesize <op> N`、`and` / `or` / `not`、括弧、`//` … `/* … */` コメント。

**完全な libyara エンジンではありません**：モジュール（`pe` / `hash` / `math` 等）、
hex の jump（`[4-6]`）やオルタネーション（`( A | B )`）は非対応で、使用すると
一致ではなく型付きエラーになります。

!!! tip "ルールの直接入力 / 一括スキャン"
    ルールを直接貼り付けたい、またはファイルやディレクトリをまとめて走査したい場合は、
    [MCP](mcp.md#tools) の `yara_scan` ツールを使ってください。ルールをテキストで渡し、
    対象にファイル・ディレクトリ・インラインのバイト列を指定できます。

## 変換レシピ { #transform-recipes }

**コード → 変換レシピの適用…** は、選択範囲（または文書全体）に対して変換を
左から右へ連結して適用します。ステップは `|`（またはスペース / カンマ）で区切り、
引数はコロンの後に書きます。例：

```
from_base64 | xor:cafe | gunzip
```

開くダイアログの 1 行の入力欄にレシピを書きます。欄の下のヒントに、エンジンが
受け付けるすべてのステップが下表と同じ分類で並ぶので、名前を覚えておく必要は
ありません。ステップを入力して **Enter**（または OK）で適用、**Esc** で
キャンセルです。

![変換レシピのダイアログとステップ一覧](assets/screenshots/convert_recipe_dialog.png){ loading=lazy }

### 使えるステップの一覧

| 分類 | ステップ |
| --- | --- |
| 引数あり | `xor:<鍵>`（繰り返し鍵で各バイトを XOR）、`rc4:<鍵>`（RC4 暗号化 / 復号）、`rot:<n>`（n 文字のシーザーシフト） |
| 引数なし | `from_hex`、`to_hex`、`rot13`、`not`（全ビット反転）、`reverse`（バイト順を反転）、`base58.encode` / `base58.decode`、`base85.encode` / `base85.decode` |
| 短い別名 | `from_base64` = `base64.decode`、`to_base64` = `base64.encode`、`from_base32` / `to_base32`、`gunzip` = `gzip.decompress`、`gzip` = `gzip.compress`、`url_decode` / `url_encode` |
| エンコード | `base64.encode`、`base64url.encode`、`url.encode`、`qp.encode`、`html.encode`、`unicode.escape`、`rot13`、`base32.encode` |
| デコード | `base64.decode`、`base64url.decode`、`url.decode`、`qp.decode`、`html.decode`、`unicode.unescape`、`jwt.decode`、`base32.decode` |
| データ | `timestamp.unixToIso`、`timestamp.unixMsToIso`、`timestamp.filetimeToIso`、`timestamp.isoToUnix`、`csv.toJson`、`json.toCsv`、`csv.toTsv`、`tsv.toCsv`、`csv.toMarkdown`、`csv.toHtml`、`csv.toSql`、`json.toYaml`、`yaml.toJson`、`json.toToml`、`toml.toJson`、`json.toIni`、`ini.toJson`、`json.minify`、`json.pretty`、`html.toMarkdown` |
| 圧縮 | `gzip.compress`、`gzip.decompress`、`zlib.compress`、`zlib.decompress`、`bzip2.compress`、`bzip2.decompress` |

エンコード / デコード / データ / 圧縮の行は[ファイル変換](convert.md)の
コンバーターで、どのコンバーター ID もそのままステップとして使えます。鍵の引数
（`xor:` / `rc4:`）は、`cafe` や `deadbeef` のように 16 進に見える場合は 16 進
バイト列、それ以外は文字列として扱います。`hex:cafe` / `str:cafe` と書けば明示
できます。

### 例

エンコードされたテキストを選択して（または文書全体に）適用します：

| レシピ | 内容 |
| --- | --- |
| `from_base64` | Base64 をデコード |
| `from_base64 \| gunzip` | Base64 デコード後に gzip 解凍 |
| `from_hex \| xor:5a` | 16 進デコード後、各バイトを `0x5A` で XOR |
| `from_hex \| xor:hex:deadbeef` | 16 進デコード後、繰り返し鍵 `DE AD BE EF` で XOR |
| `from_base64 \| rc4:str:s3cret` | Base64 デコード後、鍵 `s3cret` で RC4 復号 |
| `rot13` | ROT13（`rot:3` はシーザー暗号の 3 シフト） |
| `base58.decode` | Base58 文字列をデコード（ウォレットアドレス／鍵など） |
| `jwt.decode` | JWT をデコード済みヘッダー／ペイロードの JSON に展開 |
| `from_base64 \| xor:cafe \| from_base64` | 二重エンコード＋XOR のペイロードを剥がす |

結果は選択範囲（または文書）を**1 回の undo 単位**で置き換えるので、`Ctrl+Z` で
レシピ全体を取り消せます。途中のステップが失敗した場合（例：不正な Base64）は、
どのステップで失敗したかを表示し、テキストは変更しません。

## エントロピー

[HEX / バイナリエディタ](hex.md)のエントロピーマップとオーバービューストリップが、
ファイルを領域ごとに Shannon エントロピー（0–8 bit/byte）で表示するため、パックや
暗号化された領域が目立ちます。

## シークレットの検出

**コード → シークレットを検出** は、文書内の疑わしいシークレット（API キー、
トークン、秘密鍵、資格情報）を行番号付きで一覧し、行をクリックするとジャンプします。
**編集 → 選択範囲をマスク** は選択テキストを伏せ字にします。
