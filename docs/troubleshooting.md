# Troubleshooting

## Activation problems

- Make sure you enter the **same email** you purchased with, and the key exactly as
  in the email (`ZUME-XXXX-XXXX-XXXX-XXXX`).
- *"Already active on 3 devices"* - deactivate an old device
  (**Help → Deactivate This Device** there), or contact support.
- *"Could not be verified"* - check your internet connection and try again. If your
  subscription was cancelled or refunded, the license is no longer valid.

## Where logs and settings are

- **Windows:** `%LOCALAPPDATA%\\ZumeEditor\\` (logs under `logs\\`).
- **macOS:** `~/Library/Application Support/ZumeEditor/`.
- **Linux:** `~/.local/share/ZumeEditor/` (or `$XDG_DATA_HOME`).

## Reset settings

Close the app and move or delete `settings.json` in the folder above; the app
recreates defaults on next launch.

## Reporting a bug

Please [open an issue](https://github.com/zume-editor-dev/zume-editor-help/issues)
with your OS, the app version (**Help → About**), steps to reproduce, and the
relevant lines from the log if you can. See also [Support](support.md).
