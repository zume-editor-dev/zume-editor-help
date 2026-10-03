# CLI (zume-cli)

A command-line companion, `zume-cli`, ships alongside the app (next to the
executable on Windows, in `bin/` on Linux, and in the app folder on macOS). It
reuses the editor's own engines, so results match the GUI.

```text
zume-cli <command> [arguments] [flags]
```

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

## `aikey` - gen-AI API keys (for the MCP hub)

Manage the named API keys that the [MCP](mcp.md) `http_request` tool injects. Keys
are kept in the OS credential store, never in a file.

```text
zume-cli aikey set <name> [value]     # store a key (prints: stored '<name>')
zume-cli aikey list                   # list key names, one per line
zume-cli aikey remove <name>          # delete a key (prints: removed '<name>')
```
If you omit `value` for `set`, the key is read from standard input so it is not
left in your shell history.

## `dropbox` - Dropbox files

```text
zume-cli dropbox login <app_key>        # authorize and save the token
zume-cli dropbox ls [path]              # list a folder (root by default)
zume-cli dropbox get <remote> <local>   # download a file
zume-cli dropbox put <local> <remote>   # upload a file
zume-cli dropbox logout                 # forget the saved token
```

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

## Examples

```text
zume-cli find TODO src/main.cpp
zume-cli search "fn\s+\w+" src --regex --ignore-case
zume-cli replace http:// https:// config.ini
zume-cli aikey set openai            # then paste the key at the prompt
zume-cli geo csv2geojson points.csv
zume-cli geo geohash 35.68 139.76 8
zume-cli geo reproject --from 4326 --to 3857 area.geojson
```
