# MBTiles 1.3

## Abstract

MBTiles is a specification for storing tiled map data in
[SQLite](http://sqlite.org/) databases for immediate usage and for transfer.
MBTiles files, known as **tilesets**, must implement the specification below
to ensure compatibility with devices.

## Database Specifications

Tilesets are expected to be valid SQLite databases of
[version 3.0.0](http://sqlite.org/formatchng.html) or higher.
Only core SQLite features are permitted; tilesets **cannot require extensions**.

MBTiles databases can optionally use [the officially assigned magic number](http://www.sqlite.org/src/artifact?ci=trunk&filename=magic.txt)
to be easily identified as MBTiles.

## Database

Note: the schemas outlined are meant to be followed as interfaces.
SQLite views that produce compatible results are equally valid.
For convenience, this specification refers to tables and virtual
tables (views) as tables.

### Metadata

#### Schema

The database is required to contain a table or view named `metadata`.

This table must yield exactly two columns named `name` and
`value`. A typical create statement for the `metadata` table:

    CREATE TABLE metadata (name text, value text);

It is common to create an index for this table:

    CREATE UNIQUE INDEX name on metadata (name);

#### Content

The metadata table is used as a key/value store for settings. Two keys are **required**:

* `name`: The plain-English name of the tileset.
* `format`: The file format of the tile data: `png`, `jpg`, `pbf`, or `mvt`.

`pbf` and `mvt` both refer to zlib-deflated vector tile data in
[Mapbox Vector Tile](https://github.com/mapbox/vector-tile-spec/) format.
Historically, vector tilesets have been written with `pbf` as the `format`,
but version 2 and beyond of the Mapbox Vector Tile spec prefer `mvt`.

Four rows in `metadata` are **suggested** and, if provided, may enhance performance:

* `bounds`: The maximum extent of the rendered map area. Bounds must define an
  area covered by all zoom levels. The bounds are represented in `WGS:84` -
  latitude and longitude values, in the OpenLayers Bounds format -
  **left, bottom, right, top**. Example of the full earth: `-180.0,-85,180,85`.
* `center`: The longitude, latitude, and zoom level of the default view of the map.
  Example: `-122.1906,37.7599,11`
* `minzoom`: The lowest zoom level for which the tileset provides data
* `maxzoom`: The highest zoom level for which the tileset provides data

Other optional rows encode more information about the tileset:

* `attribution`: An attribution string, which explains in English (and HTML) the sources of
  data and/or style for the map.
* `description`: A description of the tileset's content as plain text.
* `type`: `overlay` or `baselayer`
* `version`: The version of the tileset, as a plain number.
  This refers to a revision of the tileset itself, not of the MBTiles specification.
* `scheme`: `tms`, to make the tiling scheme explicit

One other row is **required** if the `format` is `pbf` or `mvt`:

* `json`: Lists the layers that appear in the vector tiles and the names and types of
  the attributes of features that appear in those layers. See [below](#vector-tileset-metadata) for more detail.

Several additional keys are supported for tilesets that implement
[UTFGrid-based interaction](https://github.com/mapbox/utfgrid-spec).

### Tiles

#### Schema

The database is required to contain a table named `tiles`.

The table must yield four columns named `zoom_level`, `tile_column`,
`tile_row`, and `tile_data`. A typical create statement for the `tiles` table:

    CREATE TABLE tiles (zoom_level integer, tile_column integer, tile_row integer, tile_data blob);

It is common to create an index for this table:

    CREATE UNIQUE INDEX tile_index on tiles (zoom_level, tile_column, tile_row);

#### Content

The tiles table contains tiles and the values used to locate them.
The `zoom_level`, `tile_column`, and `tile_row` columns follow the
[Tile Map Service Specification](http://wiki.osgeo.org/wiki/Tile_Map_Service_Specification) in
their construction, but in a restricted form:

**The [global-mercator](http://wiki.osgeo.org/wiki/Tile_Map_Service_Specification#global-mercator) (aka Spherical Mercator) profile is assumed**

Note that the Y axis is reversed from the coordinate system commonly used in the URLs
to request individual tiles, so the tile commonly referred to as 11/327/791 is inserted as
`zoom_level` 11, `tile_column` 327, and `tile_row` 1256, since 1256 is 2^11 - 1 - 791.

The `tile_data blob` column contains raw image or vector tile data in binary.

A subset of file formats are permitted:

* `png`
* `jpg`
* `pbf`
* `mvt`

### Grids

_See the [UTFGrid specification](https://github.com/mapbox/utfgrid-spec) for
implementation details of grids and interaction metadata itself: the MBTiles
specification is only concerned with storage._

#### Schema

The database can have optional tables named `grids`, `grid_data`.

The `grids` table must yield four columns named `zoom_level`, `tile_column`,
`tile_row`, and `grid`. A typical create statement for the `grids` table:

    CREATE TABLE grids (zoom_level integer, tile_column integer, tile_row integer, grid blob);

The `grid_data` table must yield five columns named `zoom_level`, `tile_column`,
`tile_row`, `key_name`, and `key_json`. A typical create statement for the `grid_data` table:

    CREATE TABLE grid_data (zoom_level integer, tile_column integer, tile_row integer, key_name text, key_json text);

#### Content

The `grids` table contains UTFGrid data, gzip compressed.

The `grid_data` table contains grid key to value mappings, with values encoded
as JSON objects.

## Vector tileset metadata

As mentioned above, Mapbox Vector Tile tilesets must include a `json` row in the `metadata` table
to summarize what layers are available in the tiles and what attributes are available for the
features in those layers.

The row contains a single JSON object with a `vector_layers` key, whose value is an array of layers.
Each layer is a JSON object with the following keys:

* `id`: The layer ID, used for a [CartoCSS selector](https://tilemill-project.github.io/tilemill/docs/guides/selectors/) or [Mapbox GL Style layer](https://www.mapbox.com/mapbox-gl-style-spec/#layers).
* `description`: A human-readable description of the layer's contents (or empty string if not available).
* `fields`: A JSON object whose keys and values are the names and types of attributes available in this layer. The type should be `Number`, `Boolean`, or `String`.

The layer object may also contain these keys:

* `minzoom`: The lowest zoom level whose tiles this layer appears in.
* `maxzoom`: The highest zoom level whose tiles this layer appears in.

### Example

A vector tileset that contains United States counties and primary roads from [TIGER](https://www.census.gov/geo/maps-data/data/tiger-line.html) might
have the following metadata table:

* `name`: `TIGER 2016`
* `format`: `mvt`
* `bounds`: `-179.231086,-14.601813,179.859681,71.441059`
* `center`: `-84.375000,36.466030,5`
* `minzoom`: `0`
* `maxzoom`: `5`
* `attribution`: `United States Census`
* `description`: `US Census counties and primary roads`
* `type`: `overlay`
* `version`: `2`
* `scheme`: `tms`
* `json`:
```
    {
        "vector_layers": [
            {
                "id": "tl_2016_us_county",
                "description": "Census counties",
                "minzoom": 0,
                "maxzoom": 5,
                "fields": {
                    "ALAND": "Number",
                    "AWATER": "Number",
                    "COUNTYFP": "String",
                    "GEOID": "String",
                    "MTFCC": "String",
                    "NAME": "String",
                    "NAMELSAD": "String",
                    "STATEFP": "String"
                }
            },
            {
                "id": "tl_2016_us_primaryroads",
                "description": "Census primary roads",
                "minzoom": 0,
                "maxzoom": 5,
                "fields": {
                    "FULLNAME": "String",
                    "LINEARID": "String",
                    "MTFCC": "String",
                    "RTTYP": "String"
                }
            }
        ]
    }
```
