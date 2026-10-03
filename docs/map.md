# MAP (Geo Data Conversion)

Convert and measure geospatial data without leaving the editor
(**Tools → Geo**).

## Format conversion

Convert between **GeoJSON, WKT, KML, GPX and CSV** (point data). A neutral model is
used internally, so any format converts to any other.

## Coordinates & transforms

- **DMS &harr; decimal** degrees, and **geohash** encode / decode.
- **Slippy-map tiles** (z/x/y, Bing quadkey, bounding box).
- **Japan Plane Rectangular CS** (平面直角座標系, JGD2011, zones 1-19) &harr; lat/lon.

## Measurement

- Geodesic **length** and **area** of GeoJSON / WKT geometry.

All of this is also available from the command line - see
[CLI → geo](cli.md).
