# MBTiles Specification

MBTiles is a specification for storing arbitrary tiled map data in
[SQLite](http://sqlite.org/) databases for immediate usage and for efficient transfer.
MBTiles files, known as **tilesets**, must implement the specification below
to ensure compatibility with devices.

# Versions

* [1.3](https://github.com/mapbox/mbtiles-spec/blob/master/1.3/spec.md)
* [1.2](https://github.com/mapbox/mbtiles-spec/blob/master/1.2/spec.md)
* [1.1](https://github.com/mapbox/mbtiles-spec/blob/master/1.1/spec.md)
* [1.0](https://github.com/mapbox/mbtiles-spec/blob/master/1.0/spec.md)

# Concept

MBTiles is a compact, restrictive specification. It supports only
tiled data, including vector or image tiles and interactivity grid tiles. Only the
Spherical Mercator projection is supported for presentation (tile display),
and only latitude-longitude coordinates are supported for metadata such
as bounds and centers.

It is a minimum specification, only specifying the ways in which data
must be retrievable. Thus MBTiles files can internally compress and optimize
data, and construct views that adhere to the MBTiles specification.

Unlike [Spatialite](http://www.gaia-gis.it/spatialite/), GeoJSON,
and Rasterlite, MBTiles is not raw data storage. It is storage
for tiled data, like rendered map tiles.

One MBTiles file represents a single tileset, optionally including grids
of interactivity data. Multiple tilesets (layers, or maps in other
terms) can be represented by multiple MBTiles files.

## UTFGrid

The MBTiles specification previously contained the
[UTFGrid specification](https://github.com/mapbox/utfgrid-spec).
It was removed in version 1.2 and moved into its own specification
with synced version numbers, so MBTiles 1.2 is compatible with
UTFGrid 1.2. The specs integrate but do not require each other
for compliance.

# [Implementations](https://github.com/mapbox/mbtiles-spec/wiki/Implementations).

# License

The text of this specification is licensed under a
[Creative Commons Attribution 3.0 United States License](http://creativecommons.org/licenses/by/3.0/us/).
However, the use of this spec in products and code is entirely free:
there are no royalties, restrictions, or requirements.

# Authors

* Eric Fischer (ericfischer)
* Konstantin Kaefer (kkaefer)
* Tom MacWright (tmcw)
* Justin Miller (incanus)
* Paul Norman (pnorman)
* Blake Thompson (flippmoke)
* Will White (willwhite)
