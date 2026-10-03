# CLI（zume-cli）

コマンドライン用の補助ツール `zume-cli` を同梱しています（Windowsは実行ファイルの隣、
Linuxは `bin/`、macOSはアプリフォルダー内）。エディター本体のエンジンを再利用するため、
結果は GUI と一致します。

```text
zume-cli <command> [引数] [フラグ]
```

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

## `aikey` - 生成AIの API キー（MCPハブ用）

[MCP](mcp.md) の `http_request` ツールが付与する、名前付き API キーを管理します。キーは OS の
資格情報ストアに保存され、ファイルには残りません。

```text
zume-cli aikey set <name> [value]     # 保存（出力: stored '<name>'）
zume-cli aikey list                   # 名前を1行ずつ一覧
zume-cli aikey remove <name>          # 削除（出力: removed '<name>'）
```
`set` で `value` を省略すると、標準入力から読み取ります（シェル履歴に残さないため）。

## `dropbox` - Dropbox のファイル

```text
zume-cli dropbox login <app_key>        # 認可してトークンを保存
zume-cli dropbox ls [path]              # フォルダー一覧（既定はルート）
zume-cli dropbox get <remote> <local>   # ダウンロード
zume-cli dropbox put <local> <remote>   # アップロード
zume-cli dropbox logout                 # 保存トークンを破棄
```

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

## 例

```text
zume-cli find TODO src/main.cpp
zume-cli search "fn\s+\w+" src --regex --ignore-case
zume-cli replace http:// https:// config.ini
zume-cli aikey set openai            # 続けてプロンプトでキーを貼り付け
zume-cli geo csv2geojson points.csv
zume-cli geo geohash 35.68 139.76 8
zume-cli geo reproject --from 4326 --to 3857 area.geojson
```
