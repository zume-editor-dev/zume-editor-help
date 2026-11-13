# インストール

Zume Editor は **Windows 10/11・macOS・Linux** で動作します。

[Zume Editor をダウンロード](https://smolt-stage.com/home/ja/download.html){ .md-button .md-button--primary }

各種ダウンロード（Windows のポータブル ZIP / インストーラー、macOS の `.dmg`、
Linux の AppImage / DEB / RPM）は
[ダウンロードページ](https://smolt-stage.com/home/ja/download.html) にあります。

## Windows

- **Microsoft Store**（推奨） - *Zume Editor* を検索してインストール。更新は自動。
- **スタンドアロン** - ポータブルZIPとインストーラーも
  [ダウンロードページ](https://smolt-stage.com/home/ja/download.html) で配布しています。

### ポータブルモード（USB メモリから実行）

ポータブル ZIP はインストール不要です。USB メモリを含む任意の場所に展開して、
`zume-editor.exe` を実行するだけで動きます。ただし既定では、設定やセッションは
起動した PC 側のユーザープロファイルに保存されます。

設定ごと持ち歩きたい場合は、`zume-editor.exe` の隣に **`portable.txt`** という
名前の空ファイルを作成してください。設定・セッション・回復チェックポイント・
最近使ったファイル・known hosts・ログ・ライセンスがすべて exe の隣の
**`data`** フォルダーに保存されるようになり、USB メモリごと持ち運べます。
`portable.txt` を削除すると、PC ごとの保存に戻ります。

## macOS

- [ダウンロードページ](https://smolt-stage.com/home/ja/download.html) から
  **`.dmg`** をダウンロードし、開いて *アプリケーション* へドラッグ。
  署名・公証（notarize）済みです。

## Linux

- **AppImage** - ダウンロードして `chmod +x` で実行。インストール不要。
- **DEB / RPM** - パッケージマネージャ（`apt` / `dnf`・`rpm`）でインストール。

## 動作環境

- Windows 10 version 1809 以降 / Windows 11
- macOS（新しめのバージョン）
- GTK 4 が動く 64bit Linux デスクトップ

## 初回起動

初回起動時に **30日間の無料体験** が始まります（アカウント不要）。ご利用を続ける
場合は公式サイトでサブスクを購入し、アプリ内で認証してください。詳しくは
[ライセンス認証](activation.md)。
