# MBTiles Specification for PostgreSQL/PostGIS

MBTiles-pg is a possible specification for storing tiled map data in
[PostgreSQL] databases for immediate usage and for transfer.
MBTiles images, known as **tilesets**, must implement the specification below
to ensure compatibility with devices.

# Concept

MBTiles is a compact, restrictive specification. It supports only
tiled data, including image tiles and interactivity grid tiles. Only the
Spherical Mercator projection is supported for presentation - tile display -
and only latitude-longitude coordinates are supported for metadata such
as bounds and centers.

It is a minimum specification - only specifying the ways in which data
must be retrievable. Thus MBTiles files can internally compress and optimize
data, and construct views that adhere to the MBTiles specification.

Unlike [Spatialite](http://www.gaia-gis.it/spatialite/), GeoJSON,
and Rasterlite, MBTiles is not raw data storage - it is storage
for presentational data, like rendered map tiles.

One MBTiles file represents a single tileset, optionally including grids
of interactivity data. Multiple tilesets - layers, or maps in other terms,
can be represented by multiple MBTiles files.

# [Implementations](https://github.com/mapbox/mbtiles-spec/wiki/Implementations).

# License

The text of this specification is licensed under a
[Creative Commons Attribution 3.0 United States License](http://creativecommons.org/licenses/by/3.0/us/).
However, the use of this spec in products and code is entirely free:
there are no royalties, restrictions, or requirements.

# Based on work by the following authors

* Tom MacWright (tmcw)
* Will White (willwhite)
* Konstantin Kaefer (kkaefer)
* Justin Miller (incanus)
