# CLI (zume-cli)

A command-line companion, `zume-cli`, ships alongside the app (next to the
executable on Windows, in `bin/` on Linux, and in the app folder on macOS).

## Text

```text
zume-cli find <pattern> <file>
zume-cli search <pattern> <dir> [glob]
zume-cli replace <pattern> <repl> <file>
```

Flags: `--regex`, `--ignore-case`. Results match the editor's own engine.

## Geo

```text
zume-cli geo geojson2wkt <file>      # and wkt2geojson, geojson2kml, kml2geojson,
zume-cli geo geojson2gpx <file>      #     gpx2geojson, geojson2csv, csv2geojson
zume-cli geo length <file>           # geodesic length / area
zume-cli geo geohash <lat> <lon>     # and unhash, jpr, unjpr, tile, dms
```

Exit codes: `0` success, `1` error, `2` usage. See [MAP](map.md) for the geo
features in the GUI.
