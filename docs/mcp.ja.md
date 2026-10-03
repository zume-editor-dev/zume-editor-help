# MCP（AI連携）

Zume Editor は **`zume-mcp`** という別プログラムを同梱しています。これは
**MCP（Model Context Protocol）サーバー** として動作し、対応する AI クライアント
（Claude Desktop や Claude Code など）が起動して、エディター本体の実績あるエンジン経由で
ファイルの読み取り・調査・編集を行えます。

`zume-mcp` は stdio（標準入出力）上で JSON-RPC 2.0 を話します。ウィンドウは持たず、自身で
ネットワークポートを開くこともありません。アプリと同じ場所にインストールされます
（Windowsは実行ファイルの隣、Linuxは `bin/`、macOSはアプリフォルダー内）。

## ツール一覧

**ファイル / テキスト / Hex / 検索 / 差分**（ローカル・ネットワーク不要）:

| ツール | 内容 |
| --- | --- |
| `read_file_range` | ファイルの指定バイト範囲を読む（超大容量でも可） |
| `file_info` | ファイルの有無とバイトサイズ |
| `search_in_file` | リテラル / 正規表現検索。オフセット・行付きで一致を返す |
| `extract_strings` | 印字可能ASCIIの連なりをバイトオフセット付きで抽出 |
| `hex_dump` | 指定バイト範囲の 16進 + ASCII ダンプ |
| `find_bytes` | 16進バイト列の全出現オフセット（先頭 64 MiB） |
| `patch_bytes` | 指定オフセットのバイトを**その場で上書き**（ファイルを変更） |
| `replace_in_file` | リテラル / 正規表現の一致を全置換し**保存**（ファイルを変更） |
| `detect_encoding` | 文字コードと改行コードの判定 |
| `diff_files` | 2ファイルの行差分（ハンクごとの追加 / 削除行） |

**生成AIハブ:**

| ツール | 内容 |
| --- | --- |
| `list_credentials` | ローカル保存された API キーの**名前**一覧 |
| `http_request` | 任意の HTTPS API を、保存済みキーを**名前指定**で付与して呼び出す |

`http_request` では AI は保存済みキーの「名前」を指定するだけで、**秘密値は `zume-mcp`
側で付与され、AI 側（MCP境界）には渡りません**。キーは `zume-cli aikey set` で保存します
（[CLI](cli.md) 参照）。

## 使用例

MCP クライアントの起動コマンドに `zume-mcp` を指定します。例として、JSON 設定を使う
クライアント（Claude Desktop など）の場合:

```json
{
  "mcpServers": {
    "zume-editor": {
      "command": "zume-mcp"
    }
  }
}
```

Windows で `zume-mcp` が `PATH` に無い場合はフルパスを指定します。例:
`"command": "C:\\Program Files\\Zume Editor\\zume-mcp.exe"`

接続後は、AI に次のように依頼できます:

- 「`C:\\logs\\huge.log` の末尾 2&nbsp;KB を読んで」→ `read_file_range`
- 「`firmware.bin` から `DE AD BE EF` のバイト列を探して」→ `find_bytes`
- 「`legacy.txt` の文字コードと改行は？」→ `detect_encoding`
- 「`config.ini` の `http://` を全部 `https://` に置換して」→ `replace_in_file`
- 「`before.json` と `after.json` の差分を見せて」→ `diff_files`

## 利用上の注意

!!! warning "一部のツールはディスク上のファイルを変更します"
    `patch_bytes` と `replace_in_file` は**ファイルをその場で変更 / 上書き**します。これらの
    書き込みは MCP 側からは**取り消せません**（エディター画面での編集とは異なります）。信頼
    できるファイルにのみ使い、バックアップやバージョン管理を併用してください。

- **あなたの権限で動作します。** `zume-mcp` は、あなたのアカウントが読み書きできるファイル
  をすべて操作できます。管理下の環境でのみ接続し、どのフォルダーを触らせるかに注意して
  ください。
- **外部リクエスト。** `http_request` は任意の HTTPS URL に到達できます。URL は AI が選ぶ
  ため、動作を確認してください。保存キーの値は AI に渡りませんが、リクエスト自体はあなたの
  名義で実行されます。
- **ローカル専用。** サーバー自体は何も待ち受けません（クライアントとは stdio で通信）。
  外部へ出る唯一の経路は `http_request` です。
- **テレメトリなし。** `zume-mcp` は自発的に何も送信せず、クライアントのツール呼び出しに
  応じて動作するだけです。
