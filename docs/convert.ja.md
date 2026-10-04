# ファイル変換・アーカイブ

Zume Editor は、テキストやデータを多くの形式間で変換し、Markdown / HTML を PDF
に出力し、zip / tar / 7z アーカイブの作成・展開ができます。エディタからも
`zume-cli` コマンドラインからも利用できます。

## ファイルを変換する

**Convert: Convert File…**（コマンドパレット / Tools）に全変換が一覧表示され
ます。1つ選んで保存先を指定すると、結果がそのファイルに書き出されます。入力は
アクティブなドキュメントです。

利用できる変換：

| 分類 | 変換 |
| --- | --- |
| テキストエンコード | Base64 / Base64URL エンコード・デコード、URL エンコード・デコード、HTML 実体参照 エンコード・デコード、Unicode エスケープ・アンエスケープ、ROT13、Base32 エンコード・デコード |
| トークン | JWT デコード |
| データ | CSV ↔ JSON、CSV ↔ TSV、CSV → Markdown 表、CSV → HTML、CSV → SQL、JSON ↔ YAML、JSON ↔ TOML、JSON ↔ INI、JSON 圧縮 / 整形、HTML → Markdown |
| 圧縮（単一ストリーム） | gzip・zlib・bzip2 の 圧縮 / 展開 |

!!! note
    ここでの gzip / zlib / bzip2 は 1 ファイルを 1 ストリームに圧縮します。
    複数ファイルの `.tar.gz` / `.tar.bz2` は下の[アーカイブ](#archives)を参照
    してください。

## PDF へ出力

Markdown / HTML ドキュメントは PDF として保存できます：

- **File → Export to PDF…**、変換ピッカーの **to PDF**、または CLI（下記）。
- 保存ダイアログで `.pdf` の保存先を指定します。
- 描画は各 OS 標準エンジン（Windows: Edge WebView2 ランタイム、macOS: Core
  Graphics、Linux: Pango / Cairo）を使うため、日本語などもシステムフォントで
  正しく描画されます。

## アーカイブ { #archives }

**フォルダパネル** でファイル / フォルダを右クリックして作成・展開できます：

- **ここに展開（Extract Here）** — アーカイブファイル上で、アーカイブ名の
  サブフォルダへ展開します。
- **圧縮…（Compress…）** — ファイル / フォルダ（複数選択可）上で保存先を指定。
  **選んだ拡張子で形式が決まります**。

| 形式 | 展開 | 作成 |
| --- | :---: | :---: |
| `.zip` | ✅ | ✅ |
| `.tar`、`.tar.gz` / `.tgz`、`.tar.bz2` / `.tbz2`、`.tar.xz` / `.txz` | ✅ | ✅ |
| `.7z` | ✅ | ✅ |
| `.rar` / RAR5 | ✅ | — |
| `.xz` | ✅ | — |

!!! note
    RAR は展開専用です（独自仕様のため）。暗号化された RAR / 7z には非対応です。
    非 ASCII 名のエントリは `.7z` コンテナで往復できない場合があります（ファイル
    の中身には影響しません）。

## コマンドラインから

```text
# 変換
zume-cli convert                      # 全変換を一覧
zume-cli convert <id> <in> [out]      # 変換（out 省略時は標準出力）
zume-cli convert md.toPdf   in.md  out.pdf
zume-cli convert html.toPdf in.html out.pdf

# アーカイブ
zume-cli archive list    <archive>
zume-cli archive extract <archive> [dest-dir]     # 既定: .
zume-cli archive create  <archive> <path>...      # 拡張子で形式判定
```

`create` は `.zip`、`.tar`、`.tar.gz` / `.tgz`、`.tar.bz2` / `.tbz2`、
`.tar.xz` / `.txz`、`.7z` を作成し、`extract` は `.rar` と `.xz` も読めます。
CLI の全リファレンスは[コマンドライン](cli.md)ページを参照してください。
