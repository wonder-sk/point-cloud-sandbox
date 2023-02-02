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
- STAC replacement/competitor

## File format

A virtual point cloud uses JSON file format as described in [RFC 8259](https://www.rfc-editor.org/rfc/rfc8259).

It uses `.vpc` extension (to allow easy format recognition based on the extension).

Why JSON:
- a single file
- read/write support by virtually any language (no need for extra libs)
- human readable
- easy to store optional extra metadata if needed

## Structure

### `vpc`

Specifies version number of the specification.

```json
{
  "vpc": "1.0.0"
}
```

### `files`

Contains an array of objects with details about referenced point cloud files. Each object contains the following attributes:

- `filename` - string with path to the file. It may be absolute or relative path, or http(s) URL to a remote file
- `count` - total number of points in the file
- `bbox` - an array with 6 elements, containing 3D bounding box of the data: `[xmin, ymin, zmin, xmax, ymax, zmax]`

Example:

```json
{
  "files": [
    {
      "filename": "./file1.laz",
      "count": 17063621,
      "bbox": [
        376625.001,
        5440689.0,
        625.741,
        377249.999,
        5441419.999,
        869.878
      ]
    },
    {
      "filename": "./file2.laz",
      "count": 21581101,
      "bbox": [
        376575.075,
        5441420.0,
        634.271,
        377249.999,
        5442419.999,
        965.015
      ]
    }
  ]
}
```

### `metadata`

Contains metadata about the dataset. Required attributes:

- `crs` - value is a string representing dataset's coordinate reference system in well-known text (WKT) format. Empty string if CRS is unknown, `_mix_` in case of multiple coordinate reference systems in a single dataset.


TODO:
- info about attributes?
- scaling of coordinates?

### `stats` (optional)

TODO: JSON with stats on different attributes of points.

Useful for client software to quickly identify valid ranges of data for each attribute.

### `boundary` (optional)

TODO: GeoJSON of true boundary of each referenced file (a polygon or multi-polygon geometry).

Useful for client software to give user a better understanding of the actual area covered.

### `overviews` (optional)

TODO: a list of files just like in the `files` structure defined above, but in addition to that, each file has `spacing` attribute with an estimated spacing between points. A single area may be covered by multiple overview files with different spacing.

It is assumed that overviews are thinned and merged versions of the original point cloud data.

Useful for client software to show preview of the point cloud when zoomed out, without having to open all individual files and only rely on overviews
(the same idea as with overviews of raster layers).


## Alternatives Considered

- STAC - it would require a directory of JSON files (one JSON file per LAS/LAZ/COPC file) which is impractical for our use case. Single-file STAC encoding deprecated: https://github.com/stac-extensions/single-file-stac
  - should be possible to have a relatively simple 1:1 conversion between VPC and STAC formats if needed
