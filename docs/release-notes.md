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

---

A subscription is planned for the stable (1.0) release. Your feedback shapes the
1.0 — thank you for testing the beta.
