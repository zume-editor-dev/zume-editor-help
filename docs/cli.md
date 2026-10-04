# CLI (zume-cli)

A command-line companion, `zume-cli`, ships alongside the app (next to the
executable on Windows, in `bin/` on Linux, and in the app folder on macOS). It
reuses the editor's own engines, so results match the GUI.

```text
zume-cli <command> [arguments] [flags]
```

The commands below are the **complete** set. `zume-cli` is a deliberately small
headless subset of the editor (the full tool is the app itself); there is no
`--version`, and running it with no arguments or an unknown command prints the
usage and exits with code `2`.

## Global flags

| Flag | Effect | Applies to |
| --- | --- | --- |
| `--regex` | Treat the pattern as a regular expression (default: literal) | `find`, `search`, `replace` |
| `--ignore-case` | Case-insensitive match (default: case-sensitive) | `find`, `search`, `replace` |

## Exit codes

| Code | Meaning |
| --- | --- |
| `0` | Success |
| `1` | Runtime error (prints `error: <message>` to stderr - e.g. file not found, bad regex) |
| `2` | Usage error (wrong arguments; prints usage) |

## Text commands

### `find` - search one file

```text
zume-cli find <pattern> <file> [--regex] [--ignore-case]
```
Prints one line per match as `file:line:column` (1-based), then a summary
`N match(es)`.

### `search` - recursive find in files

```text
zume-cli search <pattern> <dir> [glob] [--regex] [--ignore-case]
```
Searches `dir` recursively. `glob` optionally restricts file names (e.g. `*.cpp`).
`.git`, `build` and `node_modules` are always skipped. Prints
`file:line:column: preview` per match, then `N match(es) in M file(s)` (with
`(truncated)` if the result cap was hit).

### `replace` - replace-all in one file

```text
zume-cli replace <pattern> <replacement> <file> [--regex] [--ignore-case]
```
Replaces every match and saves the file in place (regex back-references like `$1`
work in `--regex` mode). Prints `N replacement(s)`. The file is written only when
at least one replacement was made.

## `aikey` - gen-AI API keys (for the MCP hub) {#aikey}

Manage the named API keys that the [MCP](mcp.md) `http_request` tool injects. Keys
are kept in the OS credential store, never in a file.

```text
zume-cli aikey set <name> [value]     # store a key (prints: stored '<name>')
zume-cli aikey list                   # list key names, one per line
zume-cli aikey remove <name>          # delete a key (prints: removed '<name>')
```
If you omit `value` for `set`, the key is read from standard input so it is not
left in your shell history.

## `cloud` - cloud drives {#cloud}

Work with files on **Dropbox, Google Drive, OneDrive and Box**. The refresh
token is kept in the OS credential store (never a file), and is shared with the
editor - a login here or in the app works in both.

```text
zume-cli cloud <provider> login ...          # authorize and save the token
zume-cli cloud <provider> ls [path]          # list a folder (root by default)
zume-cli cloud <provider> get <remote> <local>   # download a file
zume-cli cloud <provider> put <local> <remote>   # upload a file
zume-cli cloud <provider> logout             # forget the saved token
```

`<provider>` is `dropbox`, `googledrive`, `onedrive` or `box`. The `login`
arguments differ by provider (you supply your own OAuth app's credentials):

```text
zume-cli cloud dropbox     login <app_key>
zume-cli cloud googledrive login <client_id> <client_secret>
zume-cli cloud onedrive    login <client_id>
zume-cli cloud box         login <client_id> <client_secret>
```

Dropbox shows a code to paste back; Google / OneDrive / Box open a browser and
capture the redirect on a local port (register `http://localhost:53682/callback`
as the redirect URI in your OAuth app). `zume-cli dropbox ...` still works as an
alias for `zume-cli cloud dropbox ...`.

## `geo` - map / geospatial data

Reuses the editor's geo engine (see [MAP](map.md)). File conversions read a file
and print the result to **stdout**; coordinates may be given as decimal degrees or
DMS (e.g. `35°41'N`).

### Format conversion

```text
zume-cli geo geojson2wkt <file>
zume-cli geo wkt2geojson <file>
zume-cli geo geojson2kml <file>
zume-cli geo kml2geojson <file>
zume-cli geo geojson2gpx <file>
zume-cli geo gpx2geojson <file>
zume-cli geo geojson2csv <file>
zume-cli geo csv2geojson <file> [lonCol latCol]
```
`csv2geojson` auto-detects the longitude/latitude header columns; if it cannot,
pass the two **0-based** column indices (`lonCol latCol`).

### Measurement

```text
zume-cli geo length <file>     # geodesic length in metres  (GeoJSON or WKT input)
zume-cli geo area   <file>     # geodesic area in square metres
```

### Coordinate operations

| Command | Arguments | Output |
| --- | --- | --- |
| `geohash` | `<lat> <lon> [precision=9]` | the geohash string |
| `unhash` | `<geohash>` | `lat, lon` |
| `jpr` | `<lat> <lon> <zone>` | `northing, easting` (Japan Plane Rectangular CS, zone 1-19) |
| `unjpr` | `<X> <Y> <zone>` | `lat, lon` |
| `tile` | `<lat> <lon> <zoom>` | `z/x/y  quadkey=…  bbox=west,south,east,north` |
| `dms` | `<lat> <lon>` | decimal `lat, lon` |
| `reproject` | `--from <epsg> --to <epsg> <file.geojson>` | reprojected GeoJSON (via PROJ) |

## `convert` - file conversion

```text
zume-cli convert                      # list every converter id
zume-cli convert <id> <in> [out]      # convert a file (stdout if out is omitted)
```

`<id>` is a converter such as `base64.encode`, `csv.toJson`, `json.toYaml`,
`gzip.compress`, `bzip2.compress`, `html.toMarkdown`. Markdown / HTML to PDF has
its own ids and always needs an output file:

```text
zume-cli convert md.toPdf   notes.md   notes.pdf
zume-cli convert html.toPdf page.html  page.pdf
```

See [File conversion & archives](convert.md) for the full list and the in-editor
equivalents.

## `archive` - zip / tar / 7z archives

```text
zume-cli archive list    <archive>                 # list entries
zume-cli archive extract <archive> [dest-dir]      # extract (default: current dir)
zume-cli archive create  <archive> <path>...       # create from files / folders
```

`create` picks the format from the extension: `.zip`, `.tar`, `.tar.gz` / `.tgz`,
`.tar.bz2` / `.tbz2`, `.tar.xz` / `.txz`, `.7z`. `extract` also reads `.rar` and
`.xz` (read-only). Encrypted archives are not supported.

## Examples

```text
zume-cli find TODO src/main.cpp
zume-cli search "fn\s+\w+" src --regex --ignore-case
zume-cli replace http:// https:// config.ini
zume-cli aikey set openai            # then paste the key at the prompt
zume-cli geo csv2geojson points.csv
zume-cli geo geohash 35.68 139.76 8
zume-cli geo reproject --from 4326 --to 3857 area.geojson
zume-cli convert json.toYaml config.json config.yaml
zume-cli convert md.toPdf README.md README.pdf
zume-cli archive create site.tar.gz public/
zume-cli archive extract bundle.7z out/
```
