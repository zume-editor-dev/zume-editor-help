# Installation

Zume Editor runs on **Windows 10/11, macOS and Linux**.

[Download Zume Editor](https://smolt-stage.com/home/en/download.html){ .md-button .md-button--primary }

All downloads — the Windows portable ZIP and installer, the macOS `.dmg`, and the
Linux AppImage / DEB / RPM — are on the
[download page](https://smolt-stage.com/home/en/download.html).

## Windows

- **Microsoft Store** (recommended) - search for *Zume Editor* and install.
  Updates are delivered automatically.
- **Standalone** - a portable ZIP and an installer are also available from the
  [download page](https://smolt-stage.com/home/en/download.html).

### Portable mode (run from a USB drive)

The portable ZIP needs no installation — extract it anywhere, a USB drive
included, and run `zume-editor.exe`. By default, settings and session data are
still stored in the user profile of the PC it runs on.

To carry everything with you, create an empty file named **`portable.txt`**
next to `zume-editor.exe`. Settings, sessions, recovery checkpoints, recent
files, known hosts, logs and the license are then kept in a **`data`** folder
beside the executable instead, so the editor travels with the drive. Delete
`portable.txt` to go back to per-PC storage.

!!! warning "Portable use: features that need external software (won't run if it isn't installed)"

    The following features are **not** bundled with Zume Editor and **do not work unless the required software is installed on the PC you run it on**. Take note when launching from a USB drive on an arbitrary machine. Everything else — the editor itself, search, compare (diff), SFTP/FTP, disassembly / binary analysis and scripting (Lua) — works from the drive alone.

    | Feature | External software required | Notes |
    | --- | --- | --- |
    | Git network / advanced operations: fetch / pull / push / clone / merge / rebase / interactive rebase / worktree / LFS | A `git` command on the PC (ADR 0020) | status / diff / log / branch / stage / commit / checkout are built in via libgit2 and work offline |
    | Decompile (Ghidra): `Code: Decompile` / MCP `decompile` | Ghidra itself + a JDK (Java 17+). The `ghidra_home` setting is required | The command is disabled (off) when unset / not detected |
    | .NET (IL) decompilation: MCP `il_decompile` / `il_types` | ILSpy's `ilspycmd` (`dotnet tool install -g ilspycmd`) + `ZUME_ILSPY` | |
    | Java decompilation: MCP `java_decompile` / `java_types` | The JDK's `javap` (bytecode); for source output a CFR jar (`ZUME_CFR`) | |
    | Run Security Analyzer: `Tools: Run Security Analyzer` / MCP `analyze_run` | A user-configured external analysis tool (e.g. sanctum) (ADR 0048) | No attack code ships with the product; off until a tool is configured |

## macOS

- Download the **`.dmg`** from the
  [download page](https://smolt-stage.com/home/en/download.html), open it and
  drag Zume Editor to *Applications*. The app is signed and notarized.

## Linux

- **AppImage** - download, `chmod +x` and run; no installation needed.
- **DEB / RPM** - install with your package manager (`apt`, `dnf`/`rpm`).

## System requirements

- Windows 10 version 1809 or later / Windows 11.
- macOS (a recent version).
- A 64-bit Linux desktop with GTK 4.

## First launch

On first launch you get a **30-day free trial** - no account needed. When you are
ready, buy a subscription on the website and activate in the app. See
[Activation & Licensing](activation.md).
