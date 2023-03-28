# Virtual Point Cloud (VPC)

## Purpose

Draft specification for a file format that groups multiple point cloud files to be treated as a single dataset.

Inspired by GDAL's VRTs and PDAL's tindex.

Goals:
- load VPC in QGIS as a single point cloud layer (rather than each file as a separate map layer)
- run processing tools on a VPC as input
- simple format that's easy to read/write
- allow referencing both local and remote datasets, especially LAS/LAZ/COPC files

Non-goals:
- support other kinds of data than (georeferenced) point clouds

## File format

We are using [STAC API ItemCollection](https://github.com/radiantearth/stac-api-spec/blob/main/fragments/itemcollection/README.md) as the file format and expecting these STAC extensions:
 - [pointcloud](https://github.com/stac-extensions/pointcloud/)
 - [projection](https://github.com/stac-extensions/projection/)

Note: ItemCollection not the same thing as a [STAC Collection](https://github.com/radiantearth/stac-spec/blob/master/collection-spec/README.md). An ItemCollection is essentially a single JSON file representing a GeoJSON FeatureCollection containing STAC Items. A STAC Collection is also a JSON file,
but with a different structure and extra metadata, but more importantly, it only links to other standalone JSON files (STAC items) which
is impractical for our use case as we strongly prefer to have the whole virtual point cloud definition in a single file (for easy manipulation).

We use `.vpc` extension (to allow easy format recognition based on the extension).

Why STAC:
- it is a good fit into the larger ecosystem of data catalogs, avoiding creation of a new format
- supported natively by PDAL as well (readers.stac)
- search endpoint on STAC API servers returns the same ItemCollection that we use, so a search result can be fed directly as input
- the ItemCollection file is an ordinary GeoJSON and other clients can consume it (to at least show boundaries of individual files)

### Coordinate Reference Systems (CRS)

Each referenced file can have its own CRS and it is defined through the "projection" STAC extension - either using `proj:epsg` or `proj:wkt2` or `proj:projjson`. It is recommended that a single virtual point cloud only references files in the same CRS.

### Statistics

The STAC pointcloud extension defines optional `pc:statistics` entry with statistics for each attribute. There is a limitation that currently it does not define how to store distinct values where it makes sense and their counts (e.g. Classification attribute) - see https://github.com/stac-extensions/pointcloud/issues/5.

### Boundaries

The format requires that boundaries of referenced files are defined in WGS 84 (since GeoJSON requires that) - either as a simple 2D or 3D bounding box or as a boundary geometry (polygon / multi-polygon). In addition to that, the "projection" STAC extension allows that the bounding box or boundary geometry can be specified in native coordinate reference system - this is strongly recommended and if `proj:bbox` or `proj:geometry` are present, they will be used instead of their WGS 84 equivalents.

### Overviews (optional)

Overviews are useful for client software to show preview of the point cloud when zoomed out, without having to open all individual files and only rely on overviews
(the same idea as with overviews of raster layers).

TODO: how to define overviews? We need a list of files just like in the `assets` structure of STAC items, but in addition to that, each file has a flag it is overview, and it needs a `spacing` attribute with an estimated spacing between points. A single area may be covered by multiple overview files with different spacing.

It is assumed that overviews are thinned and merged versions of the original point cloud data.
