# MCP（AI連携）

Zume Editor は **`zume-mcp`** という別プログラムを同梱しています。これは
**MCP（Model Context Protocol）サーバー** として動作し、対応する AI クライアント
（Claude Desktop や Claude Code など）が起動して、エディター本体の実績あるエンジン経由で
ファイルの読み取り・調査・編集を行え、さらに**アプリ内に保管した API キー**を使って外部
サービスへ HTTPS で到達できます。

`zume-mcp` は stdio（標準入出力）上で JSON-RPC 2.0 を話します。ウィンドウは持たず、自身で
ネットワークポートを開くこともありません。アプリと同じ場所にインストールされます
（Windowsは実行ファイルの隣、Linuxは `bin/`、macOSはアプリフォルダー内）。

## ツール一覧

**ファイル / テキスト / Hex / 検索 / 差分**（ローカルディスク）:

| ツール | 内容 |
| --- | --- |
| `read_file_range` | ファイルの指定バイト範囲を読む（超大容量でも可） |
| `file_info` | ファイルの有無とバイトサイズ |
| `search_in_file` | リテラル / 正規表現検索。オフセット・行付きで一致を返す |
| `extract_strings` | 印字可能ASCIIの連なりをバイトオフセット付きで抽出 |
| `hex_dump` | 指定バイト範囲の 16進 + ASCII ダンプ |
| `find_bytes` | 16進バイト列の全出現オフセット（先頭 64 MiB） |
| `patch_bytes` | **既存**ファイルの指定オフセットをバイト上書き（変更） |
| `replace_in_file` | 既存ファイルのリテラル / 正規表現一致を全置換し**保存** |
| `detect_encoding` | 文字コードと改行コードの判定 |
| `diff_files` | 2ファイルの行差分（追加 / 削除行） |

**生成AIハブ:**

| ツール | 内容 |
| --- | --- |
| `list_credentials` | ローカル保存された API キーの**名前**一覧 |
| `http_request` | 任意の HTTPS API へリクエスト。保存キーで認証も可 |

## AI にできること・できないこと

期待値を明確に。AI は上記ツール経由で**のみ**動作します。

**できること:**

- あなたのアカウントが読めるファイルの**読み取り・調査**（テキスト・ログ・バイナリ）。
- 既存ファイルの**変更**: `replace_in_file`（検索置換して保存）、`patch_bytes`（バイト
  上書き）。実際の書き込みです。
- **HTTPS で外部へ送信・取得**（`http_request`、GET/POST/PUT/PATCH/DELETE/HEAD、独自
  ヘッダー・ボディ可）。**アプリ内に保管した API キーで認証**できます。キーは**名前で
  指定**し、`zume-mcp` が秘密値を `Authorization` ヘッダーに付与します。秘密値が AI 側に
  返ることは**ありません**。つまり AI は、あなたの代わりに外部 API・LLM・サービスを呼び、
  内容の送信や結果の取得ができます。

**できないこと（仕様）:**

- **ゼロからの新規ファイル作成** - ファイル系ツールは**既存**ファイルの変更のみです。
  （先にエディターで対象ファイルを作成・保存してから、AI に中身を書かせてください。）
- **データベースへのネイティブ接続** - DB 用ツールはなく、`http_request` は HTTPS 専用の
  ため、MySQL / PostgreSQL / SQL Server / Redis / MongoDB のワイヤプロトコルは話せません。
  それはエディターの [データベース](database.md) 機能で行ってください（データ源に HTTPS
  API がある場合は `http_request` で可）。
- **専用のクラウドドライブ操作** - クラウドの upload / download は、各社の HTTPS API を
  `http_request` で叩く形でのみ可能です（下記）。ワンクリックの
  [クラウドドライブ](clouddrive.md) 参照はエディター機能です。
- **`http://`・ループバック・ローカルネットワーク URL への到達** - `https://` のみ許可。

## セットアップ

MCP クライアントの起動コマンドに `zume-mcp` を指定します。JSON 設定を使うクライアント
（Claude Desktop など）の場合:

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

AI に使わせる API キーは `zume-cli aikey set <name>` で保存します
（[CLI](cli.md#aikey)）。AI 側は `list_credentials` で名前を
確認できます。

## 業務ワークフローの例 {#business-workflow}

いずれも AI への1つの依頼で、AI 自身がツールを組み合わせて実行します。

**1. コードベース全体の検索と修正**

> 「`src` 配下で `legacy_init(` を呼んでいるファイルをすべて見つけて、`init(` に変えて」

AI は `search_in_file` で該当箇所を探し、`replace_in_file` で各ファイルを修正・保存します。

**2. Web API からの情報でファイルを補完**

> 「`orders.csv` の各行について、REST API からステータスを取得して `status` 列に書いて」

AI はファイルを読み（`read_file_range`）、保存キーで認証した `http_request`
（例 `credential: "orders-api"`）で API を呼び、`replace_in_file` で書き戻します。
*（対象ファイルは既存である必要があります。）*

**3. クラウドドライブへアップロード**

> 「`report.pdf` を Dropbox の `/reports` にアップロードして」

Dropbox のトークンを保存（`zume-cli aikey set dropbox`）しておけば、AI はファイルを読み、
Dropbox のコンテンツ API へ `http_request` の `POST` を行います。*（専用クラウドツールは
なく、各社 HTTPS API 経由です。対話的な参照はエディターの
[クラウドドライブ](clouddrive.md) で。）*

**4. HTTP データ源からレコードを取得して CSV へ**

> 「分析 API から今月のサインアップを取得して `signups.csv` に入れて」

AI は HTTPS のデータ API を `http_request` で呼び、行を整形し、既存の `signups.csv` へ
`replace_in_file` で書き込みます。

!!! warning "DB には MCP から直接到達できません"
    `http_request` は HTTPS 専用のため、AI が MySQL / PostgreSQL / SQL Server / Redis /
    MongoDB サーバーへ**直接接続することはできません**。「DB に接続してテーブルを CSV に
    出力」は、エディターの [データベース](database.md) ワークスペースの仕事です（接続 →
    クエリ実行 → グリッドを CSV 保存）。DB が HTTPS のデータ API を備える場合を除き、MCP
    では行えません。

**5. 保管キーで外部 LLM / API を呼ぶ**

> 「`notes.md` を OpenAI で要約して、先頭に貼って」

一度キーを保存（`zume-cli aikey set openai`）すれば、AI は `http_request`（`url`、
`credential: "openai"`、`body`）で LLM を呼び、要約をファイルに書き込みます。キーの値は
ローカルの資格情報ストアに留まり、AI には見えません。

### AI に Lua スクリプトを書かせる（エディターのカスタマイズ） {#ai-lua}

強力な使い方: 繰り返し作業用の **Zume [Luaスクリプト](automation.md) を AI に書かせ**、
エディターで実行します。

> 「選択行の `=` を揃える Zume の Lua スクリプトを書いて、`align_equals.lua` に保存して」

AI は `zume` API を使ってスクリプトを作成し、**既存**の `align_equals.lua` に書き込みます
（先に空ファイルを作るか、返ってきたスクリプトを貼り付け）。あとは **Script → Run Lua
File** で実行し、思いどおりになるまで AI と調整します。

!!! note
    `zume-mcp` は Lua を**実行しません**。実行するのはエディター（編集はすべて取り消し
    可能）です。スクリプトはあなたが確認して実行するため、AI にエディターを拡張させつつ、
    主導権はあなたに残ります。

## 利用上の注意

!!! warning "一部のツールはディスク上のファイルを変更します"
    `patch_bytes` と `replace_in_file` は**ファイルをその場で変更 / 上書き**します。これらの
    書き込みは MCP 側からは**取り消せません**（エディター画面での編集とは異なります）。信頼
    できるファイルにのみ使い、バックアップやバージョン管理を併用してください。

!!! warning "AI はあなたのデータを外部へ送信できます"
    `http_request` を通じて、AI は保管済みキーを使い、ファイル内容などを任意の HTTPS
    エンドポイントへ送信したり、外部データを取得したりできます。サーバー自体に
    **テレメトリはなく**、自発的に通信することはありませんが、`http_request` は AI が駆動する
    実際の外部送信経路です。到達させたいキーだけを保存し、信頼できるクライアントにのみ MCP を
    接続し、AI の動作を確認してください。

- **あなたの権限で動作します。** `zume-mcp` は、あなたのアカウントが読み書きできるファイル
  をすべて操作できます。どのフォルダー・ファイルを対象にするかに注意してください。
- **HTTPS 専用・ボディ上限。** `http_request` は `https://` 以外を拒否し、応答ボディは
  1 MiB で打ち切られます（`body_truncated` で通知）。
