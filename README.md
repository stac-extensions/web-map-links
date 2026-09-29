# Web Map Links Extension Specification

- **Title:** Web Map Links
- **Identifier:** <https://stac-extensions.github.io/web-map-links/v2.0.0/schema.json>
- **Field Name Prefix:** none, but some relation types have a distinct prefix for additional data (e.g. `wms` and `wmts`)
- **Scope:** Item, Catalog, Collection
- **Extension [Maturity Classification](https://github.com/radiantearth/stac-spec/tree/master/extensions/README.md#extension-maturity):** Proposal
- **Owner**: @m-mohr

This document explains the Web Map Links Extension to the
[SpatioTemporal Asset Catalog](https://github.com/radiantearth/stac-spec) (STAC) specification.
It allows to provide links to web map services for visualization purposes.

The following services are supported:

- [3D Tiles](#3d-tiles)
- [OGC WMS](#ogc-wms)
- [OGC WMTS](#ogc-wmts)
- [PMTiles](#pmtiles)
- [TileJSON](#tilejson)
- [XYZ](#xyz)

Important resources in this extension:

- Examples:
  - [Item example](examples/item.json): Shows the basic usage of the extension in a STAC Item
  - [Collection example](examples/collection.json): Shows the basic usage of the extension in a STAC Collection
- [JSON Schema](json-schema/schema.json)
- [Changelog](./CHANGELOG.md)
- [Migration from v1](#migration-from-v1)

## Links and link templates

This extension extends the [Link Object](https://github.com/radiantearth/stac-spec/tree/master/commons/links.md#link-object)
used in all STAC entities (Catalogs, Collections, Items). It requires specific relation types to be set for the `rel` field.

Services with a fixed URL are provided as Link Objects in `links`.
Services with a templated URL (currently [XYZ](#xyz)) are provided as Link Template Objects in `linkTemplates`
as defined in the [Link Templates Extension](https://github.com/stac-extensions/link-templates),
which then also needs to be listed in `stac_extensions`.

**Each web map link represents exactly one layer on a map.**
If a service offers multiple layers that should be shown individually, provide multiple links.
This allows clients and other specifications to refer to each layer individually.

### Fields for all web map links

| Field Name     | Type   | Description |
| -------------- | ------ | ----------- |
| rel            | string | **REQUIRED**. The service type, see the sections below. |
| type           | string | The media type of the tiles or images the client requests (e.g. `image/png`), unless specified otherwise below. |
| title          | string | A title for the layer, e.g. for a layer switcher. |
| min_resolution | number | The finest resolution at which the layer provides useful content, see [resolution ranges](#resolution-ranges). |
| max_resolution | number | The coarsest resolution at which the layer provides useful content, see [resolution ranges](#resolution-ranges). |

An attribution field is not defined as part of this extension, but it is RECOMMENDED to provide an attribution
via the `attribution` field as defined in the [Attribution Extension](https://github.com/stac-extensions/attribution).

### Resolution ranges

`min_resolution` and `max_resolution` describe the range of resolutions in which a layer provides useful content,
e.g. because a service only provides tiles for some levels or because the data gets meaningless beyond some level.
Clients SHOULD hide the layer outside of this range.
If `min_resolution` is provided, it must be lower than or equal to `max_resolution`.

The resolution is the **distance on the ground in meters per display (CSS) pixel**, measured at the center of the
spatial extent of the STAC entity (`bbox` in Items, the first bounding box of the spatial extent in Collections).
This makes the value independent of the coordinate reference system (CRS) of the map.
Clients convert the value into the units of their map CRS, e.g. with `getPointResolution` in OpenLayers.

For example, a `max_resolution` of `1000` for data centered at 48° North converts to:

| Map CRS                    | Resolution in map units |
| -------------------------- | ----------------------- |
| EPSG:25832 (UTM zone 32N)  | ≈ 1000 m per pixel |
| EPSG:3857 (Web Mercator)   | ≈ 1000 / cos(48°) ≈ 1494 m per pixel |
| EPSG:4326 (WGS 84)         | ≈ 0.0134° per pixel horizontally, ≈ 0.0090° per pixel vertically |

The ground resolution varies over large extents, so the range is approximate far from the center.
Clients MAY evaluate it at the center of the current view instead,
which is also the fallback for Catalogs and other cases where no spatial extent is available.

The data-side counterpart is `raster:spatial_resolution` of the [Raster Extension](https://github.com/stac-extensions/raster),
which is also measured in meters on the ground.
`gsd` is not suitable to derive the range as it describes the sensor, not the pixels of the data.

The available zoom levels of tiled services are provided separately (see e.g. `min_zoom` and `max_zoom` for [XYZ](#xyz)),
as they depend on the tiling scheme.

### 3D Tiles

Links to a [3D Tiles](https://docs.ogc.org/cs/18-053r2/18-053r2.html) implementation (versions 1.x).

| Field Name | Type   | Description |
| ---------- | ------ | ----------- |
| rel        | string | **REQUIRED**. Must be set to `3d-tiles`. |
| href       | string | **REQUIRED**. Link to a tileset. |
| type       | string | Recommended to be set to `application/json`. |

### OGC WMS

Links to a [OGC Web Map Service](https://www.ogc.org/standards/wms) (WMS) implementation (versions 1.3.0).
Only (tiled) "Basic WMS" is supported at this time.

| Field Name      | Type                 | Description |
| --------------- | -------------------- | ----------- |
| rel             | string               | **REQUIRED**. Must be set to `wms`. |
| href            | string               | **REQUIRED**. Link to the WMS, without any WMS specific query parameters. |
| type            | string               | The media type to be used for the image requests, e.g. `image/png` or `image/jpeg`. |
| layers          | \[string]            | **REQUIRED**. The layers that are requested together (in a single request) for this map layer. Can't be empty. |
| wms:styles      | \[string]            | The styles for the layers, in the same order as in `layers`. An empty string requests the default style of a layer. If not provided, the default styles are requested. |
| wms:transparent | boolean              | Sets whether the layers should be rendered transparent or not. Default: `false` |
| dimensions      | Map\<string, string> | Any additional dimension parameters to add to the request as query parameters (e.g. the dimensions `TIME` or `ELEVATION`). |

All `layers` are sent in a single request as comma-separated `LAYERS` parameter, and the `wms:styles` accordingly
as `STYLES` parameter. If the layers of a WMS should be shown individually, provide one link per layer.

- [More details on the mapping between WMS query parameters and the STAC fields](./wms.md)

### OGC WMTS

Links to a [OGC Web Map Tile Service](https://www.ogc.org/standards/wmts) (WMTS) implementation (versions 1.x).

| Field Name      | Type                 | Description |
| --------------- | -------------------- | ----------- |
| rel             | string               | **REQUIRED**. Must be set to `wmts`. |
| href            | string               | **REQUIRED**. Link to the WMTS, see below. |
| href:servers    | \[string]            | See [href:servers](#hrefservers) below for details. |
| type            | string               | The media type to be used for the tile requests, e.g. `image/png` or `image/jpeg`. |
| layers          | \[string]            | **REQUIRED**. The layer to show, as an array with exactly one element. |
| wmts:encoding   | string               | The WMTS request encoding, either `kvp` for [KVP](#kvp) or `rest` for [REST](#rest). Defaults to `kvp`. |
| tile_matrix_set | string               | The identifier of the TileMatrixSet from the capabilities to use. If not provided, clients choose one. |
| dimensions      | Map\<string, string> | Values for the dimensions of the layer (e.g. `Time`), see below. |

**href**:

- For KVP encoding, the `href` points to the URL of the capabilities document without any query parameters.
  So if your Capabilities can be requested from `https://example.com/geoserver/service/wmts?service=wmts&request=GetCapabilities`
  you'd provide `https://example.com/geoserver/service/wmts` as `href`.
- For REST encoding, the `href` must point to the capabilities document.
- The `href` can contain an optional server placeholder `{s}`. If `{s}` is used, the field [`href:servers`](#hrefservers) MUST be provided.

**dimensions**: For KVP encoding, the dimensions are added to the requests as key-value-pairs (i.e. query parameters).
For REST encoding, they provide the values for the dimension placeholders in the URL templates of the capabilities.

#### KVP

This describes the key-value-pair (KVP) request encoding, which uses query parameters to transmit additional parameters.

| Field Name    | Type   | Description |
| ------------- | ------ | ----------- |
| wmts:encoding | string | If provided, must be set to `kvp`. |

#### REST

This describes the REST request encoding, which provides a URL template with variables.

| Field Name    | Type             | Description |
| ------------- | ---------------- | ----------- |
| wmts:encoding | string           | **REQUIRED**. Must be set to `rest`. |
| uriTemplate   | string           | Can override the URL template from the capabilities document with a custom version that can be customized with the `variables` provided. |
| variables     | Map\<string, \*> | This object contains one key per substitution variable in the `uriTemplate`. |

`uriTemplate` and `variables` follow the definitions of the [Link Templates Extension](https://github.com/stac-extensions/link-templates).

**variables**: This object may provide a JSON Schema definition for any variable defined in the `uriTemplate`,
allowing to provide the default value for the specific STAC resource or the specification of ranges or other constraints.

The variables `{TileMatrixSet}`, `{TileMatrix}`, `{TileRow}`, `{TileCol}` do not need to be provided
explicitly in the `variables` as they can be determined from the capabilities.
Nevertheless, the corresponding variables can be provided if they should be further restricted to certain values or other constraints.

Providing a reasonable default value for all variables is strongly recommended as they
are likely to be used for a default visualization.
Additional information such as the data type of the variable, enumerations, minimum values, maximum values, etc.
can be provided for creating interactive UIs where the user can select values depending on the schema.
Note that clients may have varying capabilities to parse and handle the schemas provided for the variables.
If you want to ensure that the WMTS can be easily read, stick to very simple schemas
(e.g., enums for strings, minimum/maximum values for numbers).

Variables with a constant value should be encoded directly in the URL without using a variable.

### PMTiles

Links to a [PMTiles](https://github.com/protomaps/PMTiles/blob/main/spec/v3/spec.md) file (versions 3.x).

| Field Name | Type      | Description |
| ---------- | --------- | ----------- |
| rel        | string    | **REQUIRED**. Must be set to `pmtiles`. |
| href       | string    | **REQUIRED**. Link to a PMTiles file (usually ends with `.pmtiles`). |
| type       | string    | Recommended to be set to `application/vnd.pmtiles`. |
| layers     | \[string] | For vector tiles, the layers to show on the map by default. If not provided, it's up to the discretion of the implementation to choose a layer from the `vector_layers` in the PMTiles metadata. |

The [Tile Type](https://github.com/protomaps/PMTiles/blob/main/spec/v3/spec.md#tile-type-tt) of the
PMTiles data source can be read from the first 127 bytes of the binary header.

It is typical to assume a tile size of 256x256 display (CSS) pixels for raster tiles and 512x512 display pixels for vector tiles.
Tile sizes are not guaranteed to be uniform across an entire archive.

### TileJSON

Links to a [TileJSON](https://github.com/mapbox/tilejson-spec) document.

| Field Name | Type      | Description |
| ---------- | --------- | ----------- |
| rel        | string    | **REQUIRED**. Must be set to `tilejson`. |
| href       | string    | **REQUIRED**. Link to the valid TileJSON document. |
| type       | string    | Recommended to be set to `application/json`. |
| layers     | \[string] | For vector tiles, the layers to show on the map by default. If not provided, it's up to the discretion of the implementation to choose a layer from the `vector_layers` in the TileJSON. |

### XYZ

Links to a XYZ, also known as slippy map.
XYZ must be provided as Link Template Object in `linkTemplates`, see [Links and link templates](#links-and-link-templates).

| Field Name      | Type             | Description |
| --------------- | ---------------- | ----------- |
| rel             | string           | **REQUIRED**. Must be set to `xyz`. |
| uriTemplate     | string           | **REQUIRED**. The templated URL of the tiles, see below. |
| variables       | Map\<string, \*> | The variables in the `uriTemplate`, see below. |
| type            | string           | Recommended to be set to the media type of the tiles, usually `image/png` or `image/jpeg` for raster tiles and `application/vnd.mapbox-vector-tile` for vector tiles. |
| layers          | \[string]        | For vector tiles, the layers to show on the map by default. If not provided, it's up to the discretion of the implementation to choose a layer. |
| tile_matrix_set | string           | The tiling scheme, either the identifier of a well-known [OGC TileMatrixSet](https://docs.ogc.org/is/17-083r4/17-083r4.html) (e.g. `WorldCRS84Quad`) or a URL to a TileMatrixSet JSON document. Defaults to `WebMercatorQuad`. |
| min_zoom        | integer          | The lowest zoom level (`{z}`) for which tiles are available, in the given `tile_matrix_set`. |
| max_zoom        | integer          | The highest zoom level (`{z}`) for which tiles are available, in the given `tile_matrix_set`. |

**uriTemplate**: It MUST include the placeholders `{x}` and `{z}` and either `{y}` or `{-y}`.
`{-y}` indicates that the rows are counted from the bottom instead of the top, as in the Tile Map Service (TMS) specification.
These placeholders don't need to be described in `variables`.
It MAY include a placeholder for the server: `{s}`.
If `{s}` is used, `variables` MUST contain a variable `s` with an `enum` of at least two values.
Implementations can expand the given values into multiple URLs to send requests to multiple servers,
which can avoid request limits in web browsers.
For example, if you provide `https://{s}.example.com/{z}/{x}/{y}.png` as `uriTemplate` and `["a", "b"]` as `enum`
you can expand that to `["https://a.example.com/{z}/{x}/{y}.png", "https://b.example.com/{z}/{x}/{y}.png"]`.
All other parameters should be [hard-coded](https://github.com/stac-extensions/web-map-links/issues/2) with specific values,
e.g. the `{r}` parameter in Leaflet could be replaced by `2x`.

**min_zoom** and **max_zoom**: Clients should not request tiles outside of this range.
They may still show the layer beyond `max_zoom` by scaling up the tiles of `max_zoom`.
Use [resolution ranges](#resolution-ranges) to indicate in which range the layer should be visible.

### General

The following field applies to multiple types of web mapping services:

| Field Name   | Type      | Description |
| ------------ | --------- | ----------- |
| href:servers | \[string] | A list of replacement values for `{s}` in `href`s. |

#### href:servers

This field is used to specify a set of URLs for a web mapping library so that requests can be sent to multiple servers,
which can avoid request limits in web browsers.

The field applies to WMTS. For XYZ, the servers are provided as `enum` for the variable `s` instead.

It is **REQUIRED** if `{s}` is used in the `href` and then a list of at least 2 allowed values for the placeholder `{s}` must be provided.
If you only have a single value don't provide `href:servers` and instead hard-code the value into the href.

The implementations can expand the given values into multiple URLs.
For example, if you provide `https://{s}/example` as `href` and `href:servers` is `["a.com", "b.eu"]`
you can expand that to `["https://a.com/example", "https://b.eu/example"]`.

## Migration from v1

The migration from v1 to v2 is lossless and can be automated,
e.g. with [stac-migrate](https://github.com/stac-utils/stac-migrate).

| v1                                          | v2 |
| ------------------------------------------- | -- |
| `xyz` in `links` with `href`                | `xyz` in `linkTemplates` with `uriTemplate`, add the Link Templates Extension to `stac_extensions` |
| `href:servers` for XYZ                      | `enum` of the variable `s` |
| `wms:layers` with multiple elements         | One link per element, the comma-separated layer names in an element become the `layers` |
| `wms:styles`                                | Split in parallel to `wms:layers`, the comma-separated style names become the `wms:styles` |
| `wms:dimensions`                            | `dimensions` |
| `wmts:layer` (string)                       | `layers` with a single element |
| `wmts:layer` (array)                        | One link per element with `layers` containing the element |
| `wmts:dimensions`                           | `dimensions` |
| `type`: `application/xml` for WMTS in REST encoding | The media type of the tiles, derived from the file extension in `uriTemplate`, or removed if unknown |
| `pmtiles:layers`                            | `layers` |

## Contributing

All contributions are subject to the
[STAC Specification Code of Conduct](https://github.com/radiantearth/stac-spec/blob/master/CODE_OF_CONDUCT.md).
For contributions, please follow the
[STAC specification contributing guide](https://github.com/radiantearth/stac-spec/blob/master/CONTRIBUTING.md) Instructions
for running tests are copied here for convenience.

### Running tests

The same checks that run as checks on PR's are part of the repository and can be run locally to verify that changes are valid.
To run tests locally, you'll need `npm`, which is a standard part of any [node.js installation](https://nodejs.org/en/download/).

First you'll need to install everything with npm once. Just navigate to the root of this repository and on
your command line run:
```bash
npm install
```

Then to check markdown formatting and test the examples against the JSON schema, you can run:
```bash
npm test
```

This will spit out the same texts that you see online, and you can then go and fix your markdown or examples.

If the tests reveal formatting problems with the examples, you can fix them with:
```bash
npm run format-examples
```
