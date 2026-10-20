# Release Notes

The current version is shown in the app (**Help → About**). This is a **free
public beta** — all features are free to use until **2027-03-31**.

Downloads are on the [Installation](installation.md) page. Please report bugs and
requests on [GitHub](https://github.com/zume-editor-dev/zume-editor-help/issues).

## 0.30.0 — First public beta

The first public beta of Zume Editor for **Windows, macOS and Linux**.

**Highlights**

- **Very large files** — open, search, edit and save 10&nbsp;GB+ files with a
  near-instant first view, without loading them into RAM.
- **Hex editor** — offset / ASCII / EBCDIC columns, byte find & replace,
  templates, patches (IPS / BPS / UPS / bsdiff / VCDIFF).
- **Git client** — stage / commit, branches, a log graph, diffs, conflict
  resolution, fetch / pull / push, worktrees and LFS.
- **Terminal & SSH** shells in a dockable panel, and **remote editing** over
  SFTP / FTP / FTPS.
- **Database tools**, a two-pane **diff & merge**, and **CSV grid** editing.
- **Markdown / HTML live preview**, **format & encoding conversion**, a **CLI**
  (`zume-cli`) and an **MCP server** for AI agents.
- **Geo data conversion** (MAP): DMS, geohash, map tiles, GeoJSON / WKT /
  KML / GPX, and Japan Plane Rectangular CS.
- Everyday editing: syntax highlighting, code folding, outline, **multi-caret**,
  **column (block) mode**, search & replace, split view, macros, Lua scripts and
  session restore.

One license (for the upcoming stable release) covers all three platforms.

### Beta updates — October 2026

- **MCP for AI agents, greatly expanded** — the `zume-mcp` server now exposes
  most of the editor's engines as tools: cross-file search & replace, map-data
  conversion (geo), the conversion toolbox, zip/tar archives, binary patches,
  Markdown→HTML, an outline/symbol list, folder compare, streamed large-file
  search, Redis/MongoDB, read-only Git, cloud storage, sandboxed Lua scripting,
  and security analysis (entropy, CyberChef-style transform recipes, a YARA-rule
  subset scanner). A new sandbox (`ZUME_MCP_ROOTS` / `ZUME_MCP_READONLY`) confines
  what an agent can read or change. See [MCP](mcp.md#tools).
- **Markdown preview tables** — GFM pipe tables now render as a bordered grid,
  with **bold / italic / code / links inside cells** and per-column alignment.
  HTML tables render in the preview on all platforms.
- **Processing cursor** — a busy cursor is shown while a very large file is
  opening and while an in-file search or replace runs.
- **Dialog polish** — long dialog titles no longer wrap or overlap the close
  button, the Theme dialog's **X** button and header hover were fixed, and the
  Settings dialog scrollbar shows a grab cursor.
- **Stability** — fixed a crash that could occur after using the Terminal / SSH
  panel.
- Various smaller fixes across all platforms.

### Beta update — 2026-10-15 (refreshed builds)

The Windows, macOS and Linux packages were rebuilt from the same tree; the
version stays 0.30.0. Re-download from the Installation page to pick them up.

- **The transform-recipe dialog lists every step** — Code → Apply Convert
  Recipe… now shows the whole step list under its field: the keyed primitives
  (`xor:` / `rc4:` / `rot:`), the argument-free ones, the short names and every
  converter id by group, so there is nothing to memorise. The short names the
  documentation uses (`from_base64`, `to_base64`, `from_base32`, `to_base32`,
  `gunzip`, `gzip`, `url_decode`, `url_encode`) are accepted now — before, the
  engine rejected them as unknown steps. See
  [Transform recipes](security.md#transform-recipes).
- **YARA Scan in the editor** — Code → YARA Scan… takes a rules file, scans
  the active document and lists each match in the results panel; a row jumps
  to its line. The recipe also gained the transform primitives (`xor`, `rot`,
  `rc4`, `base58` / `base85`, hex) that were MCP-only before.
- **Strikethrough in the preview** — `~~text~~` and `<del>` / `<s>` /
  `<strike>` now render struck through in the Markdown / HTML preview
  (Windows and Linux; macOS draws them as plain text for now).
- **macOS** — the image was rebuilt with the fixes above and the latest
  macOS-side polish.

### Beta update — 2026-10-21 (refreshed builds)

Rebuilt from the same tree (still 0.30.0) with a large reverse-engineering
batch and floating panels. Re-download from the Installation page to pick them
up.

- **Reverse engineering** — a new **Code** menu suite reads executables
  structurally and never runs them: **Executable Info** (PE / ELF / Mach-O —
  headers, sections with entropy, imports / exports, resources, hardening),
  **Disassemble Executable** (Capstone: x86 / 64, ARM, Thumb, AArch64),
  **Cross-References** and function recovery, and optional decompilers that shell
  out to tools you install — **Ghidra**, **ILSpy** (.NET) and **javap + CFR**
  (Java). New **Objective-C / Swift Interface** recovery reads a Mach-O's
  runtime metadata (classes, methods, ivars, properties, Swift types and
  demangled symbols) — most useful on macOS / iOS binaries. The MCP server
  exposes the same engines (`pe_info`, `elf_info`, `macho_info`, `disassemble`,
  `xrefs`, `functions`, `decompile`). See [Security analysis](security.md).
- **Floating panels** — any dock panel (Terminal, results, FTP / SFTP / SSH,
  compare) can be floated into its own window — to put a Terminal on a second
  monitor, for instance — and re-docked, with its live content still updating.
  Right-click a panel's tab for **Float Tab** and **Move to Right / Bottom**.
  See [Getting Started](getting-started.md#panels-docking-moving-and-floating).
- **macOS** — the reverse-engineering suite and floating panels were verified on
  the macOS build; the image was rebuilt with the latest Mac-side polish.
- Various smaller fixes across all platforms.

---

A subscription is planned for the stable (1.0) release. Your feedback shapes the
1.0 — thank you for testing the beta.
