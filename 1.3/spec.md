# MBTiles 1.3

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in
this document are to be interpreted as described in [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt).

## Abstract

MBTiles is a specification for storing tiled map data in
[SQLite](http://sqlite.org/) databases for immediate usage and for transfer.
MBTiles files, known as **tilesets**, MUST implement the specification below
to ensure compatibility with devices.

## Compatibility

*This section is informative and does not add requirements to implementations.*

Because views may be used to produce the MBTiles schema, two implementations
may store tiles with different internal details, meaning one implementation
may not be able to add to an existing file.

As a container format, MBTiles can store any tiled data, so data can be stored
that an implementation cannot do anything with.

Relying on metadata keys not defined in the specification can cause
compatibility problems.

## Database Specifications

Tilesets SHALL be valid SQLite databases of
[version 3.0.0](http://sqlite.org/formatchng.html) or higher.
Only core SQLite features are permitted; tilesets SHALL NOT require extensions.

MBTiles databases MAY use [the officially assigned magic number](http://www.sqlite.org/src/artifact?ci=trunk&filename=magic.txt)
to be easily identified as MBTiles.

## Database

Note: the schemas outlined are meant to be followed as interfaces.
SQLite views that produce compatible results MAY be used instead.
For convenience, this specification refers to tables and virtual
tables (views) as tables.

## Charset

All text in `text` columns of tables in an mbtiles tileset MUST be encoded as UTF-8.

### Metadata

#### Schema

The database MUST contain a table or view named `metadata`.

This table or view MUST yield exactly two columns of type `text`, named `name` and
`value`. A typical create statement for the `metadata` table:

    CREATE TABLE metadata (name text, value text);

#### Content

The metadata table is used as a key/value store for settings. It MUST contain these two rows:

* `name` (string): The human-readable name of the tileset.
* `format` (string): The file format of the tile data: `pbf`, `jpg`, `png`, `webp`, or an [IETF media type](https://www.iana.org/assignments/media-types/media-types.xhtml) for other formats.

`pbf` as a `format` refers to gzip-compressed vector tile data in
[Mapbox Vector Tile](https://github.com/mapbox/vector-tile-spec/) format.

The `metadata` table SHOULD contain these four rows:

* `bounds` (string of comma-separated numbers): The maximum extent of the rendered map area. Bounds must define an
  area covered by all zoom levels. The bounds are represented as `WGS 84`
  latitude and longitude values, in the OpenLayers Bounds format
  (left, bottom, right, top). For example, the `bounds` of the full Earth, minus the poles, would be:
  `-180.0,-85,180,85`.
* `center` (string of comma-separated numbers): The longitude, latitude, and zoom level of the default view of the map.
  Example: `-122.1906,37.7599,11`
* `minzoom` (number): The lowest zoom level for which the tileset provides data
* `maxzoom` (number): The highest zoom level for which the tileset provides data

The `metadata` table MAY contain these four rows:

* `attribution` (HTML string): An attribution string, which explains the sources of
  data and/or style for the map.
* `description` (string): A description of the tileset's content.
* `type` (string): `overlay` or `baselayer`
* `version` (number): The version of the tileset.
  This refers to a revision of the tileset itself, not of the MBTiles specification.

If the `format` is `pbf`, the `metadata` table MUST contain this row:

* `json` (stringified JSON object): Lists the layers that appear in the vector tiles and the names and types of
  the attributes of features that appear in those layers. See [below](#vector-tileset-metadata) for more detail.

The `metadata` table MAY contain additional rows for tilesets that implement
[UTFGrid-based interaction](https://github.com/mapbox/utfgrid-spec) or for
other purposes.

### Tiles

#### Schema

The database MUST contain a table named `tiles`.

The table MUST contain three columns of type `integer`, named `zoom_level`, `tile_column`,
`tile_row`, and one of type `blob`, named `tile_data`.
A typical `create` statement for the `tiles` table:

    CREATE TABLE tiles (zoom_level integer, tile_column integer, tile_row integer, tile_data blob);

The database MAY contain an index for efficient access to this table:

    CREATE UNIQUE INDEX tile_index on tiles (zoom_level, tile_column, tile_row);

#### Content

The tiles table contains tiles and the values used to locate them.
The `zoom_level`, `tile_column`, and `tile_row` columns MUST encode the location
of the tile, following the
[Tile Map Service Specification](http://wiki.osgeo.org/wiki/Tile_Map_Service_Specification),
with the restriction that
the [global-mercator](http://wiki.osgeo.org/wiki/Tile_Map_Service_Specification#global-mercator) (aka Spherical Mercator) profile MUST be used.

Note that in the TMS tiling scheme, the Y axis is reversed from the "XYZ" coordinate system commonly used in the URLs
to request individual tiles, so the tile commonly referred to as 11/327/791 is inserted as
`zoom_level` 11, `tile_column` 327, and `tile_row` 1256, since 1256 is 2^11 - 1 - 791.

The `tile_data` column MUST contain the raw binary image or vector tile data
for the associated tile as a `blob`.

### Grids

_See the [UTFGrid specification](https://github.com/mapbox/utfgrid-spec) for
implementation details of grids and interaction metadata itself: the MBTiles
specification is only concerned with storage._

#### Schema

The database MAY have tables named `grids` and `grid_data`.

The `grids` table MUST contain three columns of type `integer`, named `zoom_level`, `tile_column`,
and `tile_row`, and one of type `blob`, named `grid`.
A typical create statement for the `grids` table:

    CREATE TABLE grids (zoom_level integer, tile_column integer, tile_row integer, grid blob);

The `grid_data` table MUST contain three columns of type `integer`, named `zoom_level`, `tile_column`,
and `tile_row`, and two of type `text`, named `key_name`, and `key_json`.
A typical create statement for the `grid_data` table:

    CREATE TABLE grid_data (zoom_level integer, tile_column integer, tile_row integer, key_name text, key_json text);

#### Content

The `grids` table, if present, MUST contain UTFGrid data, compressed in `gzip` format.

The `grid_data` table, if present, MUST contain grid key to value mappings, with values encoded
as JSON objects.

## Vector tileset metadata

As mentioned above, Mapbox Vector Tile tilesets MUST include a `json` row in the `metadata` table
to summarize what layers are available in the tiles and what attributes are available for the
features in those layers.

The `json` row, if present, MUST contain the UTF-8 string representation of a JSON object.

### Vector_layers

The JSON object in the `json` row MUST contain a `vector_layers` key, whose value is an array of JSON objects.
Each of those JSON objects describes one layer of vector tile data, and MUST contain the following key-value pairs:

* `id` (string): The layer ID, which is referred to as the `name` of the layer in the [Mapbox Vector Tile spec](https://github.com/mapbox/vector-tile-spec/tree/master/2.1#41-layers).
* `fields` (object): A JSON object whose keys and values are the names and types of attributes available in this layer.
Each type MUST be the string `"Number"`, `"Boolean"`, or `"String"`.
Attributes whose type varies between features SHOULD be listed as `"String"`.

Each layer object MAY also contain the following key-value pair:

* `description` (string): A human-readable description of the layer's contents.

Each layer object MAY also contain the following key-value pair:

* `minzoom` (number): The lowest zoom level whose tiles this layer appears in.
* `maxzoom` (number): The highest zoom level whose tiles this layer appears in.

The `minzoom` MUST be greater than or equal to the tileset's `minzoom`,
and the `maxzoom` MUST be less than or equal to the tileset's `maxzoom`.

These keys are used to describe the situation where different sets of vector layers
appear in different zoom levels of the same tileset, for example in a case where
a "minor roads" layer is only present at high zoom levels.

### Tilestats

The JSON object in the `json` row MAY also contain a `tilestats` key, whose value is an object in the "geostats"
format documented in the [mapbox-geostats](https://github.com/mapbox/mapbox-geostats#output-the-stats)
repository. Like the `vector_layers`, it lists the tileset's layers and the attributes found
within each layer, but also gives sample values for each attribute and the range of values for
numeric attributes.

### Example

A vector tileset that contains United States counties and primary roads from [TIGER](https://www.census.gov/geo/maps-data/data/tiger-line.html) might
have the following metadata table:

* `name`: `TIGER 2016`
* `format`: `pbf`
* `bounds`: `-179.231086,-14.601813,179.859681,71.441059`
* `center`: `-84.375000,36.466030,5`
* `minzoom`: `0`
* `maxzoom`: `5`
* `attribution`: `United States Census`
* `description`: `US Census counties and primary roads`
* `type`: `overlay`
* `version`: `2`
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
                    "GEOID": "String",
                    "MTFCC": "String",
                    "NAME": "String"
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
        ],
        "tilestats": {
            "layerCount": 2,
            "layers": [
                {
                    "layer": "tl_2016_us_county",
                    "count": 3233,
                    "geometry": "Polygon",
                    "attributeCount": 5,
                    "attributes": [
                        {
                            "attribute": "ALAND",
                            "count": 6,
                            "type": "number",
                            "values": [
                                1000508839,
                                1001065264,
                                1001787870,
                                1002071716,
                                1002509543,
                                1003451714
                            ],
                            "min": 82093,
                            "max": 376825063576
                        },
                        {
                            "attribute": "AWATER",
                            "count": 6,
                            "type": "number",
                            "values": [
                                0,
                                100091246,
                                10017651,
                                100334057,
                                10040117,
                                1004128585
                            ],
                            "min": 0,
                            "max": 25190628850
                        },
                        {
                            "attribute": "GEOID",
                            "count": 6,
                            "type": "string",
                            "values": [
                                "01001",
                                "01003",
                                "01005",
                                "01007",
                                "01009",
                                "01011"
                            ]
                        },
                        {
                            "attribute": "MTFCC",
                            "count": 1,
                            "type": "string",
                            "values": [
                                "G4020"
                            ]
                        },
                        {
                            "attribute": "NAME",
                            "count": 6,
                            "type": "string",
                            "values": [
                                "Abbeville",
                                "Acadia",
                                "Accomack",
                                "Ada",
                                "Adair",
                                "Adams"
                            ]
                        }
                    ]
                },
                {
                    "layer": "tl_2016_us_primaryroads",
                    "count": 12509,
                    "geometry": "LineString",
                    "attributeCount": 4,
                    "attributes": [
                        {
                            "attribute": "FULLNAME",
                            "count": 6,
                            "type": "string",
                            "values": [
                                "1- 80",
                                "10",
                                "10-Hov Fwy",
                                "12th St",
                                "14 Th St",
                                "17th St NE"
                            ]
                        },
                        {
                            "attribute": "LINEARID",
                            "count": 6,
                            "type": "string",
                            "values": [
                                "1101000363000",
                                "1101000363004",
                                "1101019172643",
                                "1101019172644",
                                "1101019172674",
                                "1101019172675"
                            ]
                        },
                        {
                            "attribute": "MTFCC",
                            "count": 1,
                            "type": "string",
                            "values": [
                                "S1100"
                            ]
                        },
                        {
                            "attribute": "RTTYP",
                            "count": 6,
                            "type": "string",
                            "values": [
                                "C",
                                "I",
                                "M",
                                "O",
                                "S",
                                "U"
                            ]
                        }
                    ]
                }
            ]
        }
    }

```
