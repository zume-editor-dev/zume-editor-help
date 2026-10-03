# CLI（zume-cli）

コマンドライン用の補助ツール `zume-cli` を同梱しています（Windowsは実行ファイルの隣、
Linuxは `bin/`、macOSはアプリフォルダー内）。

## テキスト

```text
zume-cli find <pattern> <file>
zume-cli search <pattern> <dir> [glob]
zume-cli replace <pattern> <repl> <file>
```

フラグ：`--regex`、`--ignore-case`。結果はエディター本体のエンジンと一致します。

## 地理（geo）

```text
zume-cli geo geojson2wkt <file>      # 他に wkt2geojson, geojson2kml, kml2geojson,
zume-cli geo geojson2gpx <file>      #     gpx2geojson, geojson2csv, csv2geojson
zume-cli geo length <file>           # 測地的な距離 / 面積
zume-cli geo geohash <lat> <lon>     # 他に unhash, jpr, unjpr, tile, dms
```

終了コード：`0` 成功 / `1` エラー / `2` 使い方。GUIでの地理機能は [地図](map.md)。
