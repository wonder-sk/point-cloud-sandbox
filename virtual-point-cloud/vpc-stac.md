
# VPC as a STAC API ItemCollection

https://github.com/radiantearth/stac-api-spec/blob/main/fragments/itemcollection/README.md

A sample item collection: https://usgs-lidar-stac.s3-us-west-2.amazonaws.com/ept/item_collection.json

## good stuff

- reusing STAC spec, no custom format
- can be opened as a vector layer too to see boundaries
- search results from STAC API could be directly used
- supported in PDAL >= 2.5


## problems

- it does not have a custom extension defined (typically .json is used) - confusion with ordinary vector GeoJSON
  - we could define our own extension (e.g. stick to .vpc)
  
- bounding box / boundary geometry of items is in wgs84
  - use "projection" extension: proj:bbox + proj:geometry
  - but proj extension is optional - what if the source does not provide this info? (or it only provides CRS, not bbox/geometry)
  
- 3d crs definitions like EPSG:1234+5678 - proj:epsg requires int
  - we can only provide proj:wkt2 and/or proj:projjson - and the compound code would get extracted from there
  
- we can't expect global properties (e.g. total point count, box, crs, stats)
  - we could extract them when loading a file
  
- stats for point clouds are not defined
  - we could start our own, then hopefully have it standardized - https://github.com/stac-extensions/pointcloud/issues/5
  
- overview point clouds
  - have them as a sidecar file? or list them among features, but with special attribute?

- there is no "global" version, in theory all items could have different versions/extensions
  - we can require a particular version and pointcloud+proj extensions

- dealing with point clouds without CRS - we need wgs84 coordinates for geojson
  - we can either not transform coordinates (violating wgs84 requirement) or use a dummy transform (placing it in a wrong location)
