# クラウドドライブ

クラウドストレージ上のファイルを開いて編集できます。

- **Dropbox**
- **Google Drive**
- **OneDrive**
- **Box**

**Remote → CloudDrive** からアカウントを接続し、各プロバイダーの安全なサインイン
（OAuth）でログインします。Zume Editor がクラウドのパスワードを見ることはありません。
サイドバーでドライブを参照し、ファイルをタブに開いて、保存でアップロードします。

!!! note
    クラウドドライブは個人向けクラウドストレージ用です。自前サーバーには
    [リモートファイル（SFTP / FTP / FTPS）](remote.md) をお使いください。

## OAuth アプリの資格情報はご自身で用意します

Zume Editor はクラウドのクライアントを内蔵していません。各プロバイダーを初めて
接続するときに、**ご自身の OAuth アプリの資格情報**（Client ID、多くのプロバイダー
では Client Secret も）を入力します。アプリは各社の開発者コンソールで一度だけ作成
します（無料・数分）。取得したトークンは **OS の資格情報ストア** に保存され（ファイル
には残らず）、`zume-cli` とも共有されます。

### リダイレクト URI

Google Drive / OneDrive / Box は、Zume Editor が起動するローカルサーバーへ
リダイレクトしてサインインを完了します。アプリに次の URI を **完全一致** で登録して
ください：

```text
http://localhost:53682/callback
```

ここの不一致が最頻出のエラー（`redirect_uri_mismatch`）です。Dropbox は代わりに
コード貼り付け方式で、リダイレクト URI は不要です。

## 初回の接続

1. **Remote → CloudDrive → Connect to &lt;プロバイダー&gt;**。
2. **Client ID**、続けて **Client Secret** を入力（Dropbox はアプリキーのみ、
   OneDrive は Client ID のみ）。
3. ブラウザが開き各社の同意画面が表示されます。許可してください。（Dropbox は
   表示されるコードをエディターに貼り付けます。）
4. Zume Editor が `localhost:53682` でリダイレクトを受け取り（**約3分以内**）、
   トークンを保存します。サイドバーにドライブが表示されます。

2回目以降は、保存トークンが有効な限りブラウザ認証は不要です
（**Remote → CloudDrive → Disconnect**、CLI の `logout` で解除）。

## Client ID / Client Secret の取得方法

### Google Drive

1. [Google Cloud Console](https://console.cloud.google.com/) を開き、プロジェクト
   を作成（または選択）。
2. **APIとサービス → ライブラリ** で **Google Drive API** を有効化。
3. **APIとサービス → OAuth 同意画面** を設定（User Type は **外部**。*テスト中*
   の間は自分の Google アカウントを **テストユーザー** に追加）。
4. **APIとサービス → 認証情報 → 認証情報を作成 → OAuth クライアント ID**、
   アプリケーションの種類は **ウェブアプリケーション**。
5. **承認済みのリダイレクト URI** に `http://localhost:53682/callback` を追加。
6. 作成後、**クライアント ID** と **クライアント シークレット** をコピー。

### Dropbox

1. [Dropbox App Console](https://www.dropbox.com/developers/apps) →
   **Create app** → **Scoped access**、Full Dropbox か App folder を選択。
2. **Permissions** タブでファイルのスコープ（読み取り・書き込み）を有効化。
3. **Settings** タブの **App key** をコピー。（Dropbox はコード貼り付け方式のため、
   リダイレクト URI もシークレットも不要です。）

### OneDrive

1. [Azure Portal](https://portal.azure.com/) → **アプリの登録** → **新規登録**。
2. リダイレクト URI に `http://localhost:53682/callback` を追加（モバイル/
   デスクトップ＝パブリッククライアント。シークレットは使いません）。
3. 委任アクセス許可 **Files.ReadWrite** を付与。
4. **アプリケーション (クライアント) ID** をコピー。

### Box

1. [Box Developer Console](https://app.box.com/developers/console) →
   **Create New App → Custom App → User Authentication (OAuth 2.0)**。
2. リダイレクト URI を `http://localhost:53682/callback` に設定。
3. 構成画面の **Client ID** と **Client Secret** をコピー。

## コマンドラインから

```text
zume-cli cloud googledrive login <client_id> <client_secret>
zume-cli cloud dropbox     login <app_key>
zume-cli cloud onedrive    login <client_id>
zume-cli cloud box         login <client_id> <client_secret>

zume-cli cloud <provider> ls [path]
zume-cli cloud <provider> get <remote> <local>
zume-cli cloud <provider> put <local> <remote>
zume-cli cloud <provider> logout
```

CLI も同じループバックサインインを使い、保存トークンをエディターと共有します。
詳しくは[コマンドライン](cli.md)を参照してください。
