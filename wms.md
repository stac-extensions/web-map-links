# OGC WMS

## Mapping between STAC and the WMS GetMap parameters

| Request Parameter           | Mandatory | Description                                                  | Mapping / Implementation guide                               |
| --------------------------- | --------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| VERSION=1.3.0               | Yes       | Request version.                                             | Always `1.3.0` - We assume everyone supports 1.3.0 today.    |
| REQUEST=GetMap              | Yes       | Request name.                                                | Always `GetMap`.                                             |
| LAYERS=layer_list           | Yes       | Comma-separated list of one or more map layers.              | via `wms:layers`, one layer in the mapping library per array element |
| STYLES=style_list           | Yes       | Comma-separated list of one rendering style per requested layer. | via `wms:styles`, 1:1 mapping to `wms:layers`                |
| CRS=namespace:identifier    | Yes       | Coordinate reference system.                                 | determined by mapping library, usually EPSG:3857             |
| BBOX=minx,miny,maxx,maxy    | Yes       | Bounding box corners (lower left, upper right) in CRS units. | determined by mapping library                                |
| WIDTH=output_width          | Yes       | Width in pixels of map picture.                              | determined by mapping library                                |
| HEIGHT=output_height        | Yes       | Height in pixels of map picture.                             | determined by mapping library                                |
| FORMAT=output_format        | Yes       | Output format of map.                                        | via `type`                                                   |
| TRANSPARENT=TRUE\|FALSE     | No        | Background transparency of map (default=FALSE).              | via `wms:transparent`                                        |
| BGCOLOR=color_value         | No        | Hexadecimal red-green-blue colour value for the background color (default=0xFFFFFF). | not supported, some libraries may support setting it via `wms:dimensions` |
| EXCEPTIONS=exception_format | No        | The format in which exceptions are to be reported by the WMS (default=XML). | determined by mapping library                                |
| TIME=time                   | No        | Time value of layer desired.                                 | via `wms:dimensions`                                         |
| ELEVATION=elevation         | No        | Elevation of layer desired.                                  | via `wms:dimensions`                                         |
| Other sample dimension(s)   | No        | Value of other dimensions as appropriate.                    | via `wms:dimensions`                                         |

Columns 1-3 are coming from the [OGC WMS specification, version 1.3.0](https://portal.ogc.org/files/?artifact_id=14416), chapter 7.3.2.
