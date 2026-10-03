# トラブルシューティング

## 認証がうまくいかない

- 購入時と **同じメールアドレス**、およびメール記載どおりのキー
  （`ZUME-XXXX-XXXX-XXXX-XXXX`）を入力してください。
- *「すでに3台で有効」* - いずれかの旧端末で解除（**Help → Deactivate This Device**）
  するか、サポートへご連絡ください。
- *「検証できませんでした」* - インターネット接続を確認して再試行してください。解約
  または返金済みの場合、そのライセンスは無効です。

## ログ・設定の保存場所

- **Windows:** `%LOCALAPPDATA%\\ZumeEditor\\`（ログは `logs\\`）
- **macOS:** `~/Library/Application Support/ZumeEditor/`
- **Linux:** `~/.local/share/ZumeEditor/`（または `$XDG_DATA_HOME`）

## 設定のリセット

アプリを終了し、上記フォルダーの `settings.json` を移動または削除してください。次回
起動時に既定値で再作成されます。

## 不具合の報告

[Issue を作成](https://github.com/zume-editor-dev/zume-editor-help/issues) し、OS・
アプリのバージョン（**Help → About**）・再現手順・可能ならログの該当行を添えてください。
[サポート](support.md) も参照してください。
