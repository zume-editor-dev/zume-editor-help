# MCP (AI Integration)

Zume Editor ships a separate program, **`zume-mcp`**, that acts as an
**MCP (Model Context Protocol) server**. A compatible AI client (such as Claude
Desktop or Claude Code) launches it and can then read, inspect and edit files
through the editor's own, tested engines.

`zume-mcp` speaks JSON-RPC 2.0 over stdio (standard input / output) - it has no
window and opens no network port of its own. It is installed alongside the app
(next to the executable on Windows, in `bin/` on Linux, inside the app folder on
macOS).

## Tools

**Files, text, hex, search, diff** (local, no network):

| Tool | What it does |
| --- | --- |
| `read_file_range` | Read a bounded byte range of a file (works on very large files) |
| `file_info` | Whether a file exists, and its size in bytes |
| `search_in_file` | Literal or regex search; returns matches with offsets / lines |
| `extract_strings` | Printable-ASCII runs with their byte offsets |
| `hex_dump` | A hex + ASCII dump of a byte range |
| `find_bytes` | Every offset of a hex byte pattern (first 64 MiB) |
| `patch_bytes` | Overwrite bytes at an offset **in place** (modifies the file) |
| `replace_in_file` | Replace all literal / regex matches and **save** (modifies the file) |
| `detect_encoding` | A file's text encoding and line ending |
| `diff_files` | A line diff of two files (added / removed lines per hunk) |

**Gen-AI hub:**

| Tool | What it does |
| --- | --- |
| `list_credentials` | List the **names** of API keys stored locally |
| `http_request` | Call any HTTPS API, injecting a stored key **by name** |

With `http_request`, the AI names a stored key; the secret value is added to the
request by `zume-mcp` and **never crosses the MCP boundary** to the AI. Store keys
with `zume-cli aikey set` (see [CLI](cli.md)).

## Usage example

Point your MCP client at the `zume-mcp` command. For example, in a client that
uses a JSON config (such as Claude Desktop):

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

Once connected, you can ask the AI things like:

- "Read the last 2&nbsp;KB of `C:\\logs\\huge.log`." → `read_file_range`
- "Find the bytes `DE AD BE EF` in `firmware.bin`." → `find_bytes`
- "What encoding and line ending does `legacy.txt` use?" → `detect_encoding`
- "Replace every `http://` with `https://` in `config.ini`." → `replace_in_file`
- "Diff `before.json` and `after.json`." → `diff_files`

## Usage notes & cautions

!!! warning "Some tools change files on disk"
    `patch_bytes` and `replace_in_file` **modify / overwrite files in place**, and
    these writes are **not undoable** from the MCP side (unlike edits made in the
    editor window). Only let the AI work on files you trust, and keep backups or use
    version control.

- **It runs with your permissions.** `zume-mcp` can read and write any file your
  user account can. Only connect it in environments you control, and be mindful of
  which folders you ask the AI to touch.
- **Outbound requests.** `http_request` can reach any HTTPS URL. The AI chooses the
  URL, so review what it does; the stored key's value is never revealed to the AI,
  but the request is still made on your behalf.
- **Local only.** The server itself listens on nothing - it communicates with the
  client over stdio. Its only outbound network path is `http_request`.
- **No telemetry.** `zume-mcp` sends nothing on its own; it only acts on the tool
  calls the client makes.
