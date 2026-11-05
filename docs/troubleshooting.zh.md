# 故障排查

## 激活问题

- 确保你输入的是购买时使用的 **相同邮箱**，以及与邮件中完全一致的
  密钥（`ZUME-XXXX-XXXX-XXXX-XXXX`）。
- *"Already active on 3 devices"* —— 停用一台旧设备
  （在该设备上执行 **Help → Deactivate This Device**），或联系支持。
- *"Could not be verified"* —— 检查你的网络连接并重试。如果你的
  订阅已被取消或退款，许可证将不再有效。

## 日志和设置的位置

- **Windows:** `%LOCALAPPDATA%\\ZumeEditor\\`（日志位于 `logs\\` 下）。
- **macOS:** `~/Library/Application Support/ZumeEditor/`。
- **Linux:** `~/.local/share/ZumeEditor/`（或 `$XDG_DATA_HOME`）。

## 重置设置

关闭应用，然后移动或删除上述文件夹中的 `settings.json`；应用
会在下次启动时重新创建默认设置。

## 报告 bug

请[提交 issue](https://github.com/zume-editor-dev/zume-editor-help/issues)，
并附上你的操作系统、应用版本（**Help → About**）、复现步骤，以及
（如果可以的话）日志中的相关行。另请参阅 [支持](support.md)。
