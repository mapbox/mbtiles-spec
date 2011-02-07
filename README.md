# MBTiles Specification

MBTiles is a specification for storing tiled map data in  [SQLite](http://sqlite.org/) databases for immediate usage and for transfer. MBTiles files, known as **tilesets**, must implement the specification below to ensure compatibility with devices.

# Versions

* [1.1](https://github.com/mapbox/mbtiles-spec/blob/master/1.1/spec.md)
* [1.0](https://github.com/mapbox/mbtiles-spec/blob/master/1.0/spec.md)

# Changelog

## 1.1

* `name='format'` row **required** in `metadata` table.
* `name='bounds'` row suggested in `metadata` table.

# Implementations

[Implementing software is tracked on the Wiki](https://github.com/mapbox/mbtiles-spec/wiki/Implementations).

# Authors

* Tom MacWright (tmcw)
* Will White (willwhite)
* Justin Miller (icanus)
