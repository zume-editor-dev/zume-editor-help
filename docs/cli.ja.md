# CLI（zume-cli）

コマンドライン用の補助ツール `zume-cli` を同梱しています（Windowsは実行ファイルの隣、
Linuxは `bin/`、macOSはアプリフォルダー内）。エディター本体のエンジンを再利用するため、
結果は GUI と一致します。

!!! warning "Microsoft Store（MSIX）版について"
    **Microsoft Store 版**では `zume-cli` を**コマンドラインから利用できません**。
    ファイルは保護された `C:\Program Files\WindowsApps\…` 配下にインストールされ
    （更新のたびに変わるバージョン別パスで、`PATH` にも登録されません）、
    `zume-cli` を直接実行・スクリプト化できないためです。Windows で CLI や
    [MCP サーバー](mcp.md)を使うには、代わりに**ポータブル ZIP**（またはインストーラー版）
    を導入し、任意の安定フォルダーに展開して、そこから `zume-cli.exe` を実行してください。

```text
zume-cli <command> [引数] [フラグ]
```

以下が**すべて**のコマンドです。`zume-cli` はエディターの機能を絞ったヘッドレス版
（本体はアプリそのもの）で、`--version` はありません。引数なし、または未知のコマンドで
実行すると usage を表示して終了コード `2` を返します。

## 共通フラグ

| フラグ | 効果 | 対象 |
| --- | --- | --- |
| `--regex` | パターンを正規表現として扱う（既定: リテラル） | `find` / `search` / `replace` |
| `--ignore-case` | 大文字小文字を区別しない（既定: 区別する） | `find` / `search` / `replace` |

## 終了コード

| コード | 意味 |
| --- | --- |
| `0` | 成功 |
| `1` | 実行時エラー（`error: <メッセージ>` を標準エラーに出力。例: ファイルなし、正規表現不正） |
| `2` | 使い方エラー（引数が不正。usage を表示） |

## テキスト系コマンド

### `find` - 1ファイルを検索

```text
zume-cli find <pattern> <file> [--regex] [--ignore-case]
```
一致ごとに `file:line:column`（1始まり）を1行ずつ出力し、最後に `N match(es)` を表示します。

### `search` - 再帰的なファイル内検索

```text
zume-cli search <pattern> <dir> [glob] [--regex] [--ignore-case]
```
`dir` を再帰的に検索します。`glob` でファイル名を限定できます（例 `*.cpp`）。`.git` /
`build` / `node_modules` は常に除外。一致ごとに `file:line:column: preview` を出力し、最後に
`N match(es) in M file(s)`（上限に達した場合は `(truncated)`）を表示します。

### `replace` - 1ファイルを全置換

```text
zume-cli replace <pattern> <replacement> <file> [--regex] [--ignore-case]
```
すべての一致を置換してその場で保存します（`--regex` では `$1` などの後方参照が使えます）。
`N replacement(s)` を出力します。1件以上置換したときのみファイルを書き込みます。

## `aikey` - 生成AIの API キー（MCPハブ用） {#aikey}

[MCP](mcp.md) の `http_request` ツールが付与する、名前付き API キーを管理します。キーは OS の
資格情報ストアに保存され、ファイルには残りません。

```text
zume-cli aikey set <name> [value]     # 保存（出力: stored '<name>'）
zume-cli aikey list                   # 名前を1行ずつ一覧
zume-cli aikey remove <name>          # 削除（出力: removed '<name>'）
```
`set` で `value` を省略すると、標準入力から読み取ります（シェル履歴に残さないため）。

## `cloud` - クラウドドライブ {#cloud}

**Dropbox / Google Drive / OneDrive / Box** のファイルを操作します。リフレッシュ
トークンは OS の資格情報ストアに保存され（ファイルには残りません）、エディターと共有
されます（どちらでログインしても双方で有効）。

```text
zume-cli cloud <provider> login ...          # 認可してトークンを保存
zume-cli cloud <provider> ls [path]          # フォルダー一覧（既定はルート）
zume-cli cloud <provider> get <remote> <local>   # ダウンロード
zume-cli cloud <provider> put <local> <remote>   # アップロード
zume-cli cloud <provider> logout             # 保存トークンを破棄
```

`<provider>` は `dropbox` / `googledrive` / `onedrive` / `box`。`login` の引数は
プロバイダーごとに異なります（ご自身の OAuth アプリの資格情報を指定）:

```text
zume-cli cloud dropbox     login <app_key>
zume-cli cloud googledrive login <client_id> <client_secret>
zume-cli cloud onedrive    login <client_id>
zume-cli cloud box         login <client_id> <client_secret>
```

Dropbox は表示されるコードを貼り付け、Google / OneDrive / Box はブラウザを開いてローカル
ポートでリダイレクトを受け取ります（OAuth アプリに `http://localhost:53682/callback` を
リダイレクト URI として登録してください）。`zume-cli dropbox ...` は
`zume-cli cloud dropbox ...` の別名として引き続き使えます。

## `geo` - 地図 / 地理空間データ

エディターの地理エンジンを再利用します（[地図](map.md) 参照）。形式変換はファイルを読み、
結果を **標準出力** に出します。座標は10進度でも DMS（例 `35°41'N`）でも指定できます。

### 形式変換

```text
zume-cli geo geojson2wkt <file>
zume-cli geo wkt2geojson <file>
zume-cli geo geojson2kml <file>
zume-cli geo kml2geojson <file>
zume-cli geo geojson2gpx <file>
zume-cli geo gpx2geojson <file>
zume-cli geo geojson2csv <file>
zume-cli geo csv2geojson <file> [lonCol latCol]
```
`csv2geojson` は経度/緯度のヘッダー列を自動判定します。判定できない場合は、**0始まり**の列
番号2つ（`lonCol latCol`）を渡してください。

### 計測

```text
zume-cli geo length <file>     # 測地的な距離（メートル）  入力は GeoJSON または WKT
zume-cli geo area   <file>     # 測地的な面積（平方メートル）
```

### 座標の操作

| コマンド | 引数 | 出力 |
| --- | --- | --- |
| `geohash` | `<lat> <lon> [precision=9]` | geohash 文字列 |
| `unhash` | `<geohash>` | `lat, lon` |
| `jpr` | `<lat> <lon> <zone>` | `northing, easting`（平面直角座標系、1〜19系） |
| `unjpr` | `<X> <Y> <zone>` | `lat, lon` |
| `tile` | `<lat> <lon> <zoom>` | `z/x/y  quadkey=…  bbox=west,south,east,north` |
| `dms` | `<lat> <lon>` | 10進の `lat, lon` |
| `reproject` | `--from <epsg> --to <epsg> <file.geojson>` | 再投影した GeoJSON（PROJ 使用） |

## `convert` - ファイル変換

```text
zume-cli convert                      # 全変換 id を一覧
zume-cli convert <id> <in> [out]      # 変換（out 省略時は標準出力）
```

`<id>` は `base64.encode`、`csv.toJson`、`json.toYaml`、`gzip.compress`、
`bzip2.compress`、`html.toMarkdown` などです。Markdown / HTML から PDF への変換は
専用 id で、出力ファイルが必須です：

```text
zume-cli convert md.toPdf   notes.md   notes.pdf
zume-cli convert html.toPdf page.html  page.pdf
```

全変換の一覧と画面からの操作は[ファイル変換・アーカイブ](convert.md)を参照して
ください。

## `archive` - zip / tar / 7z アーカイブ

```text
zume-cli archive list    <archive>                 # 一覧
zume-cli archive extract <archive> [dest-dir]      # 展開（既定: カレント）
zume-cli archive create  <archive> <path>...       # ファイル / フォルダから作成
```

`create` は拡張子で形式を判定します：`.zip`、`.tar`、`.tar.gz` / `.tgz`、
`.tar.bz2` / `.tbz2`、`.tar.xz` / `.txz`、`.7z`。`extract` は `.rar` と `.xz`
も読めます（展開のみ）。暗号化アーカイブには非対応です。

## 例

```text
zume-cli find TODO src/main.cpp
zume-cli search "fn\s+\w+" src --regex --ignore-case
zume-cli replace http:// https:// config.ini
zume-cli aikey set openai            # 続けてプロンプトでキーを貼り付け
zume-cli geo csv2geojson points.csv
zume-cli geo geohash 35.68 139.76 8
zume-cli geo reproject --from 4326 --to 3857 area.geojson
zume-cli convert json.toYaml config.json config.yaml
zume-cli convert md.toPdf README.md README.pdf
zume-cli archive create site.tar.gz public/
zume-cli archive extract bundle.7z out/
```
