# ターミナル & SSH

## ターミナルパネル

**View → Terminal Panel** で、ドッキング可能なパネルに対話シェルを開きます
（Windowsは `cmd.exe`、macOS/Linuxはログインシェル）。複数のターミナルをタブで開き、
直接入力、履歴（++up++ / ++down++）、出力の選択・コピー、++ctrl+c++ での中断が可能です。

## SSHシェル

**View → SSH Panel** で、SSH経由でリモートホストのシェルを開きます。SFTPブラウザーと
同じ接続に相乗りするか、直接接続できます。パスワード認証・鍵認証に対応
（[リモートファイル](remote.md) 参照）。

!!! note
    対話シェルのパネルであり、完全な端末エミュレーターではありません。

![下部のターミナルパネル](assets/screenshots/terminal.png){ loading=lazy }

![パネル内の SSH セッション](assets/screenshots/ssh.png){ loading=lazy }
