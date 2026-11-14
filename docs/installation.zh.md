# 安装

Zume Editor 可在 **Windows 10/11、macOS 和 Linux** 上运行。

[下载 Zume Editor](https://smolt-stage.com/home/en/download.html){ .md-button .md-button--primary }

所有下载文件（Windows 便携式 ZIP 和安装程序、macOS 的 `.dmg`、
Linux 的 AppImage / DEB / RPM）都在
[下载页面](https://smolt-stage.com/home/en/download.html)。

## Windows

- **Microsoft Store**（推荐）—— 搜索 *Zume Editor* 并安装。
  更新会自动推送。
- **独立版** —— [下载页面](https://smolt-stage.com/home/en/download.html)上也提供便携式 ZIP 和安装程序。

### 便携模式（从 U 盘直接运行）

便携式 ZIP 无需安装——解压到任意位置（包括 U 盘），直接运行
`zume-editor.exe` 即可。不过在默认情况下，设置和会话数据仍会保存到所运行
电脑的用户配置文件中。

如果想随身携带所有数据，请在 `zume-editor.exe` 旁边创建一个名为
**`portable.txt`** 的空文件。此后，设置、会话、恢复检查点、最近文件、
known hosts、日志和许可证都会保存到可执行文件旁边的 **`data`** 文件夹中，
编辑器即可随 U 盘一起移动。删除 `portable.txt` 即可恢复为按电脑保存。

!!! warning "便携使用注意：依赖外部软件的功能（未安装则无法运行）"

    以下功能**未**随 Zume Editor 一起打包，**只有在所运行的电脑上安装了相应软件时才能使用**。从 U 盘在任意电脑上启动时请特别注意。其余功能——编辑器本身、搜索、比较 (diff)、SFTP/FTP、反汇编 / 二进制分析以及脚本 (Lua)——仅凭 U 盘即可运行。

    | 功能 | 需要的外部软件 | 说明 |
    | --- | --- | --- |
    | Git 网络 / 高级操作：fetch / pull / push / clone / merge / rebase / interactive rebase / worktree / LFS | 电脑上的 `git` 命令 (ADR 0020) | status / diff / log / branch / stage / commit / checkout 由内置的 libgit2 提供，可离线使用 |
    | 反编译 (Ghidra)：`Code: Decompile` / MCP `decompile` | Ghidra 本体 + JDK (Java 17+)。需要设置 `ghidra_home` | 未设置 / 未检测到时命令将被禁用（off） |
    | .NET (IL) 反编译：MCP `il_decompile` / `il_types` | ILSpy 的 `ilspycmd`（`dotnet tool install -g ilspycmd`）+ `ZUME_ILSPY` | |
    | Java 反编译：MCP `java_decompile` / `java_types` | JDK 的 `javap`（字节码），源码输出需要 CFR jar (`ZUME_CFR`) | |
    | 运行安全分析器：`Tools: Run Security Analyzer` / MCP `analyze_run` | 用户配置的外部分析工具（如 sanctum）(ADR 0048) | 产品不随附任何攻击代码；未配置工具前为 off |

## macOS

- 从[下载页面](https://smolt-stage.com/home/en/download.html)下载 **`.dmg`**，打开它并将 Zume Editor 拖到
  *Applications* 中。该应用已签名并经过公证。

## Linux

- **AppImage** —— 下载后 `chmod +x` 并运行；无需安装。
- **DEB / RPM** —— 使用你的包管理器安装（`apt`、`dnf`/`rpm`）。

## 系统要求

- Windows 10 version 1809 或更高版本 / Windows 11。
- macOS（较新的版本）。
- 带有 GTK 4 的 64 位 Linux 桌面环境。

## 首次启动

首次启动时你会获得一个 **30 天免费试用** —— 无需账户。当你
准备好后，在网站上购买订阅并在应用内激活。请参阅
[激活与许可](activation.md)。
