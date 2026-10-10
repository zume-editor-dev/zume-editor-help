# MCP (AI Integration)

Zume Editor ships a separate program, **`zume-mcp`**, that acts as an
**MCP (Model Context Protocol) server**. A compatible AI client (such as Claude
Desktop or Claude Code) launches it and can then read, create and edit files,
query the databases you have configured in the app, and reach external services
over HTTPS using API keys you keep inside the app - all through the editor's own,
tested engines.

`zume-mcp` speaks JSON-RPC 2.0 over stdio (standard input / output) - it has no
window and opens no network port of its own. It is installed alongside the app
(next to the executable on Windows, in `bin/` on Linux, inside the app folder on
macOS).

## Tools {#tools}

**Files, text, hex, search, diff** (local disk):

| Tool | What it does |
| --- | --- |
| `read_file_range` | Read a bounded byte range of a file (works on very large files) |
| `file_info` | Whether a file exists, and its size in bytes |
| `write_file` | Create a new file, or overwrite / append an existing one (text or hex) |
| `search_in_file` | Literal or regex search; returns matches with offsets / lines |
| `extract_strings` | Printable-ASCII runs with their byte offsets |
| `hex_dump` | A hex + ASCII dump of a byte range |
| `find_bytes` | Every offset of a hex byte pattern (first 64 MiB) |
| `patch_bytes` | Overwrite bytes at an offset in an existing file |
| `replace_in_file` | Replace all literal / regex matches in an existing file and save |
| `detect_encoding` | A file's text encoding and line ending |
| `diff_files` | A line diff of two files (added / removed lines per hunk) |
| `search_in_files` | Literal or regex search across every text file under a directory |
| `replace_in_files` | Replace across a directory; previews by default, applies in one transaction |
| `search_large_file` | Streamed search over a file of any size (byte offsets + preview) |
| `compare_folders` | Compare two directory trees (same / different / left-only / right-only) |
| `list_symbols` | The definitions (functions, types, …) of a source file, with line numbers |
| `markdown_to_html` | Render Markdown (GFM tables, inline formatting) to an HTML document |

**Map data, conversion, archives, patches, scripting:**

| Tool | What it does |
| --- | --- |
| `geo_convert` | Convert map data between GeoJSON / WKT / KML / GPX / CSV |
| `geo_measure` | Geodesic length (m) / area (m²) of a GeoJSON or WKT document |
| `geo_point` | geohash, map tile + quadkey + bbox, Japan Plane Rectangular CS, DMS ⇄ decimal |
| `geo_reproject` | Reproject GeoJSON between EPSG CRSs (PROJ) |
| `convert_list` / `convert_run` | List / run the file-conversion toolbox (base64, JSON ⇄ CSV, timestamps, …) |
| `archive_list` / `archive_extract` / `archive_create` | List, extract or create zip / tar / 7z archives |
| `patch_bin` | Apply or create a binary patch (IPS / BPS / UPS / bsdiff / VCDIFF) |
| `run_script` | Run a sandboxed Lua script against a file (preview, or `save` to write it back) |

**Security analysis** (binary triage / transform / signatures):

| Tool | What it does |
| --- | --- |
| `entropy` | Shannon entropy (overall + a windowed sparkline) and the byte histogram — spot packed / encrypted regions |
| `transform` | A CyberChef-style recipe: XOR / ROT / RC4 / base58·85 / hex + the convert toolbox, composed in one pipeline |
| `yara_scan` | Scan a file, a directory or inline bytes with a practical YARA-rule subset |

**Databases** (the connections configured in the app, or a SQLite file):

| Tool | What it does |
| --- | --- |
| `db_list_connections` | List the saved SQL connections (names only; no passwords) |
| `db_schema` | List a connection's tables / views and their columns |
| `db_query` | Run SQL on a connection and return the result rows |
| `nosql_list_connections` | List the saved Redis / MongoDB connections (by name) |
| `nosql_command` | Run a Redis / MongoDB command and return the result as JSON |

**Git** (a local repository, read-only):

| Tool | What it does |
| --- | --- |
| `git_status` | The current branch and the staged / unstaged changes |
| `git_log` | The commit history (id, summary, author, time), newest first |
| `git_branches` | The local branches, with the current one and ahead / behind |
| `git_diff` | The diff of one file (staged or unstaged half) |

**Remote files (FTP / FTPS / SFTP)** (the connections configured in the app):

| Tool | What it does |
| --- | --- |
| `remote_list_connections` | List the saved FTP / FTPS / SFTP connections (by name) |
| `remote_ls` | List a remote directory |
| `remote_get` | Download a remote file (to a local file, or return its content) |
| `remote_put` | Upload a local file or inline text to a remote path |
| `remote_delete` | Delete a remote file or empty directory |
| `remote_rename` | Rename / move a remote file |
| `remote_mkdir` | Create a remote directory |

**Cloud storage** (the account you signed in to in the app, or with `zume-cli`):

| Tool | What it does |
| --- | --- |
| `cloud_ls` | List a folder on Dropbox / Google Drive / OneDrive / Box |
| `cloud_get` | Download a file from a connected cloud account |
| `cloud_put` | Upload a local file or inline text to a cloud account |

**Gen-AI hub:**

| Tool | What it does |
| --- | --- |
| `list_credentials` | List the **names** of API keys stored locally |
| `http_request` | Make an HTTPS request to any API, optionally authenticated with a stored key |

## Restrict what the AI can touch (sandbox)

Two environment variables, set on the `zume-mcp` process, scope what the file
tools may do. Both are optional — if you set neither, the tools have full access
(the default). Set them in your MCP client's server configuration (an `env`
block) to confine an agent.

| Variable | Effect |
| --- | --- |
| `ZUME_MCP_ROOTS` | A list of allowed directories (separated by `;` on Windows, `:` elsewhere). A tool is refused unless its path is inside one of these folders. Symbolic links and `..` are resolved first, so a path cannot escape. |
| `ZUME_MCP_READONLY` | `1` / `true` / `yes` / `on` makes the server read-only: every tool that would change a file (write, patch, replace, extract, create, upload, and `run_script` with `save`) is refused. Reads still work. |

```json
{
  "mcpServers": {
    "zume": {
      "command": "C:\\Tools\\Zume\\zume-mcp.exe",
      "env": { "ZUME_MCP_ROOTS": "C:\\work\\project", "ZUME_MCP_READONLY": "1" }
    }
  }
}
```

A refused request comes back as a normal tool error, not a crash. The sandbox
covers the local-file tools; the database, remote-file, cloud and `http_request`
tools have their own rules (saved connections, read-only DB profiles, HTTPS
only).

## What the AI can and cannot do

The AI acts **only** through the tools above.

**It can:**

- **Read and inspect** any file your account can read - text, logs, binaries.
- **Create, overwrite, append and byte-patch files** (`write_file`,
  `replace_in_file`, `patch_bytes`). These are real writes.
- **Query your databases.** It can list the SQL connections you configured in the
  app (`db_list_connections`), inspect a schema (`db_schema`) and run SQL
  (`db_query`) - against a saved connection by name, or a SQLite file by path.
  Passwords come from the OS credential store and are never shown to the AI.
- **Operate on remote files over FTP / FTPS / SFTP** using the connections you
  configured in the app (`remote_ls` / `remote_get` / `remote_put` /
  `remote_delete` / `remote_rename` / `remote_mkdir`). Passwords come from the OS
  credential store. For **SFTP**, the server's host key must already be trusted
  (connect once in the editor); an unknown or changed key is refused, since there
  is no way to confirm a fingerprint headlessly.
- **Send data out and pull data in over HTTPS** (`http_request`,
  GET/POST/PUT/PATCH/DELETE/HEAD), **authenticated with an API key you store
  locally**, referenced **by name** (the secret is injected by `zume-mcp` and
  never returned to the AI).

**It cannot (by design):**

- **Open a direct database connection the app has not been configured with** -
  `db_query` only uses your saved connections or a SQLite path you give it.
- **Reach a database over a non-SQL path** - `http_request` is for HTTPS APIs;
  the DB tools use the editor's SQL engines (SQLite, MySQL / MariaDB, PostgreSQL,
  SQL Server).
- **Use a dedicated cloud-drive tool** - cloud upload / download is done with the
  [`zume-cli cloud`](cli.md#cloud) command or the editor's
  [CloudDrive](clouddrive.md) browser, or by calling the provider's HTTPS API
  through `http_request`.
- **Reach `http://`, loopback or local-network URLs** - only `https://` is allowed.

## Setup

Point your MCP client at the `zume-mcp` command. For a client that uses a JSON
config (such as Claude Desktop):

```json
{
  "mcpServers": {
    "zume-editor": {
      "command": "zume-mcp"
    }
  }
}
```

On Windows, if `zume-mcp` is not on your `PATH`, use the full path, e.g.
`"command": "C:\\Program Files\\Zume Editor\\zume-mcp.exe"`.

!!! warning "Microsoft Store (MSIX) build"
    The **Microsoft Store** build cannot be used as an MCP server. Its
    `zume-mcp` is installed under the protected
    `C:\Program Files\WindowsApps\…` folder — a version-specific path that
    changes with every update and is not on your `PATH` — so an MCP client
    cannot launch it. To run the MCP server (or the [CLI](cli.md)) on Windows,
    install the **portable ZIP** (or the installer build), extract it to a
    stable folder, and point your client at that `zume-mcp.exe`, for example:

    ```json
    { "mcpServers": { "zume": { "command": "C:\\Tools\\Zume\\zume-mcp.exe" } } }
    ```

    With the Claude Code CLI:

    ```text
    claude mcp add zume -s user -- "C:\Tools\Zume\zume-mcp.exe"
    ```

The database tools read the connections you set up in the app's
[Database](database.md) workspace. Store API keys for `http_request` with
`zume-cli aikey set <name>` (see [CLI](cli.md#aikey)); the AI lists them with
`list_credentials`.

## Business workflow examples {#business-workflow}

Each example is one request you make to the AI; it chains the tools itself.

**1. Find-and-fix across a code base**

> "Find every file under `src` that still calls `legacy_init(` and change it to
> `init(`."

Uses `search_in_file` + `replace_in_file`.

**2. Export a database table to CSV**

> "Connect to the `reporting` database and export this month's `orders` to
> `orders.csv`."

The AI calls `db_query` on your saved `reporting` connection (password from the
credential store), then `write_file` to create `orders.csv` with the rows. A
connection you marked **read-only** in the app will refuse anything but a read
query.

**3. Enrich a file from a web API**

> "For each row in `orders.csv`, look up the status from our REST API and write
> the result to `orders_enriched.csv`."

The AI reads the file, calls your API with `http_request` (authenticated with a
stored key), and writes a new file with `write_file`.

**4. Upload a file to a cloud drive**

> "Upload `report.pdf` to my Dropbox `/reports` folder."

The AI reads the file and `http_request`s the Dropbox content API with a stored
token. (For interactive browsing, use [`zume-cli cloud`](cli.md#cloud)
or the editor's [CloudDrive](clouddrive.md).)

**5. Call an external LLM / API with a managed key**

> "Summarise `notes.md` with OpenAI and save the summary as `summary.md`."

Store the key once (`zume-cli aikey set openai`); the AI calls the LLM with
`http_request` and writes the result with `write_file`. The key value stays in
your local credential store and is never shown to the AI.

**6. Work with files on a server**

> "Download `/var/log/app.log` from the `staging` SFTP connection, find the
> errors, and upload a cleaned copy back as `/var/log/app.clean.log`."

The AI uses `remote_get` to download (over your saved `staging` connection),
processes the text, and `remote_put` to upload the result. The `staging` host
must already be trusted in the editor (SFTP).

### Let the AI write a Lua script (customise the editor) {#ai-lua}

A powerful pattern: have the AI **write a Zume [Lua script](automation.md)** for
a repetitive edit, then run it in the editor.

> "Write a Zume Lua script that aligns the `=` signs in the selected lines, and
> save it as `align_equals.lua`."

The AI drafts the script with the `zume` API and creates `align_equals.lua` with
`write_file`. You then run it with **Script → Run Lua File**, and iterate with
the AI until it does exactly what you want.

!!! note
    `zume-mcp` does **not** execute Lua - the editor does, where every edit is
    undoable - so you review and run the script yourself. This keeps you in
    control while still letting the AI extend the editor for you.

## Usage notes & cautions

!!! warning "Some tools change files on disk"
    `write_file`, `patch_bytes` and `replace_in_file` **create, overwrite or
    modify files**, and these writes are **not undoable** from the MCP side
    (unlike edits made in the editor window). Only let the AI work in folders you
    trust, and keep backups or use version control.

!!! warning "Database, remote and network access act on your behalf"
    `db_query` can read and (unless the connection is read-only) **write** to your
    configured databases; the `remote_*` tools can read, **upload, delete and
    rename** files on your configured FTP / SFTP servers; `http_request` can send
    data to and fetch data from any HTTPS endpoint using your stored keys.
    Passwords and key values never reach the AI, but the actions run with your
    credentials. Configure only the connections and keys you want reachable,
    connect MCP only to clients you trust, and review what the AI does.

- **It runs with your permissions.** `zume-mcp` can read and write any file your
  user account can, and reach any database you configured.
- **HTTPS only, body capped.** `http_request` rejects non-`https://` URLs, and a
  response body is truncated at 1 MiB (reported via `body_truncated`).
