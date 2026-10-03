# MCP (AI Integration)

Zume Editor ships a separate program, **`zume-mcp`**, that acts as an
**MCP (Model Context Protocol) server**. A compatible AI client (such as Claude
Desktop or Claude Code) launches it and can then read, inspect and edit files
through the editor's own, tested engines - and reach external services over HTTPS
using API keys you keep inside the app.

`zume-mcp` speaks JSON-RPC 2.0 over stdio (standard input / output) - it has no
window and opens no network port of its own. It is installed alongside the app
(next to the executable on Windows, in `bin/` on Linux, inside the app folder on
macOS).

## Tools

**Files, text, hex, search, diff** (local disk):

| Tool | What it does |
| --- | --- |
| `read_file_range` | Read a bounded byte range of a file (works on very large files) |
| `file_info` | Whether a file exists, and its size in bytes |
| `search_in_file` | Literal or regex search; returns matches with offsets / lines |
| `extract_strings` | Printable-ASCII runs with their byte offsets |
| `hex_dump` | A hex + ASCII dump of a byte range |
| `find_bytes` | Every offset of a hex byte pattern (first 64 MiB) |
| `patch_bytes` | Overwrite bytes at an offset in an **existing** file (modifies it) |
| `replace_in_file` | Replace all literal / regex matches in an existing file and **save** |
| `detect_encoding` | A file's text encoding and line ending |
| `diff_files` | A line diff of two files (added / removed lines per hunk) |

**Gen-AI hub:**

| Tool | What it does |
| --- | --- |
| `list_credentials` | List the **names** of API keys stored locally |
| `http_request` | Make an HTTPS request to any API, optionally authenticated with a stored key |

## What the AI can and cannot do

Set expectations clearly - the AI acts **only** through the tools above.

**It can:**

- **Read and inspect** any file your account can read: text, logs, and binaries
  (`read_file_range`, `search_in_file`, `extract_strings`, `hex_dump`,
  `find_bytes`, `detect_encoding`, `diff_files`).
- **Modify existing files** on disk: `replace_in_file` (find/replace and save) and
  `patch_bytes` (overwrite bytes). These are real writes.
- **Send data out and pull data in over HTTPS** with `http_request`
  (GET/POST/PUT/PATCH/DELETE/HEAD, custom headers and body), **authenticated with
  an API key you store locally in the app**. You reference the key **by name**;
  `zume-mcp` injects the secret into the request's `Authorization` header, and the
  secret value is **never** returned to the AI. So the AI can call external APIs,
  LLMs and services - uploading content and fetching results - on your behalf.

**It cannot (by design):**

- **Create a brand-new file from nothing** - the file tools modify files that
  already exist. (Create or save the target file in the editor first, then let the
  AI fill it.)
- **Connect to a database over its native protocol.** There is no DB tool, and
  `http_request` is HTTPS-only, so it cannot speak the MySQL / PostgreSQL / SQL
  Server / Redis / MongoDB wire protocols. Use the editor's [Database](database.md)
  feature for that (or, if your data source has an HTTPS API, `http_request`).
- **Use a dedicated cloud-drive tool.** Cloud upload / download is possible only by
  calling the provider's own HTTPS API through `http_request` with a stored token
  (see below); the one-click [CloudDrive](clouddrive.md) browser is an editor
  feature.
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

Store the API keys the AI may use with `zume-cli aikey set <name>` (see
[CLI](cli.md#aikey)); list them from the AI with
`list_credentials`.

## Business workflow examples {#business-workflow}

Each example is one request you can make to the AI; it chains the tools itself.

**1. Find-and-fix across a code base**

> "Find every file under `src` that still calls `legacy_init(` and change it to
> `init(`."

The AI uses `search_in_file` to locate the calls and `replace_in_file` to fix each
file (existing files, saved as it goes).

**2. Enrich a file from a web API**

> "For each row in `orders.csv`, look up the status from our REST API and write it
> into the `status` column."

The AI reads the file (`read_file_range`), calls your API with `http_request`
(authenticated with the stored key, e.g. `credential: "orders-api"`), and writes
the result back with `replace_in_file`. *(The target file must already exist.)*

**3. Upload a file to a cloud drive**

> "Upload `report.pdf` to my Dropbox `/reports` folder."

With a Dropbox token stored (`zume-cli aikey set dropbox`), the AI reads the file
and `http_request`s a `POST` to the Dropbox content API with the token. *(There is
no dedicated cloud tool - this goes through the provider's HTTPS API. For
interactive browsing use the editor's [CloudDrive](clouddrive.md).)*

**4. Pull records from an HTTP data source into a CSV**

> "Get this month's sign-ups from the analytics API and put them in `signups.csv`."

The AI `http_request`s the HTTPS data API, shapes the rows, and writes them into
the existing `signups.csv` with `replace_in_file`.

!!! warning "Databases are not reachable over MCP"
    `http_request` is HTTPS-only, so the AI **cannot** open a direct connection to
    a MySQL / PostgreSQL / SQL Server / Redis / MongoDB server. "Connect to the DB
    and export a table to CSV" is a job for the editor's [Database](database.md)
    workspace (connect, run the query, then save the grid as CSV) - not for MCP,
    unless your database is fronted by an HTTPS data API.

**5. Call an external LLM / API with a managed key**

> "Summarise `notes.md` with OpenAI and append the summary at the top."

Store the key once (`zume-cli aikey set openai`); the AI calls the LLM with
`http_request` (`url`, `credential: "openai"`, `body`) and writes the summary into
the file. The key value stays in your local credential store and is never shown to
the AI.

### Let the AI write a Lua script (customise the editor) {#ai-lua}

A powerful pattern: have the AI **write a Zume [Lua script](automation.md)** for a
repetitive edit, then run it in the editor.

> "Write a Zume Lua script that aligns the `=` signs in the selected lines, and
> save it as `align_equals.lua`."

The AI drafts the script using the `zume` API and writes it into an **existing**
`align_equals.lua` (create the empty file first, or paste the script the AI
returns). You then run it with **Script → Run Lua File**, and iterate with the AI
until it does exactly what you want.

!!! note
    `zume-mcp` does **not** execute Lua - the editor does, where every edit is
    undoable - so you review and run the script yourself. This keeps you in control
    while still letting the AI extend the editor for you.

## Usage notes & cautions

!!! warning "Some tools change files on disk"
    `patch_bytes` and `replace_in_file` **modify / overwrite files in place**, and
    these writes are **not undoable** from the MCP side (unlike edits made in the
    editor window). Only let the AI work on files you trust, and keep backups or use
    version control.

!!! warning "The AI can send your data to external services"
    Through `http_request`, the AI can transmit file contents and other data to any
    HTTPS endpoint and fetch remote data, using the API keys you have stored. The
    server itself has **no telemetry** and never phones home on its own - but
    `http_request` is a genuine outbound path that the AI drives. Store only the
    keys you want reachable, connect MCP only to clients you trust, and review what
    the AI does.

- **It runs with your permissions.** `zume-mcp` can read and write any file your
  user account can. Be mindful of which folders and files you point it at.
- **HTTPS only, body capped.** `http_request` rejects non-`https://` URLs, and a
  response body is truncated at 1 MiB (reported via `body_truncated`).
