# Web Map Links Extension Specification

- **Title:** Web Map Links
- **Identifier:** <https://stac-extensions.github.io/web-map-links/v1.3.0/schema.json>
- **Field Name Prefix:** none, but each relation type has potentially a distinct prefix for additional data (e.g. `wmts` and `xyz`)
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

## Link Object Fields

This extension only extends the [Link Object](https://github.com/radiantearth/stac-spec/tree/master/item-spec/item-spec.md#link-object)
used in all STAC entities (Catalogs, Collections, Items). It requires specific relation types to be set for the `rel` field in the
Link Object.

An attribution field is not defined as part of this extension, but it is RECOMMENDED to provide an attribution
in via the `attribution` field as defined the [Attribtion Extension](https://github.com/stac-extensions/attribution).

### 3D Tiles

Links to a [3D Tiles](https://docs.ogc.org/cs/18-053r2/18-053r2.html) implementation (versions 1.x).

| Field Name      | Type                 | Description |
| --------------- | -------------------- | ----------- |
| rel             | string               | **REQUIRED**. Must be set to `3d-tiles`. |
| href            | string               | **REQUIRED**. Link to a tileset. |
| type            | string               | Recommended to be set to `application/json`. |

### OGC WMS

Links to a [OGC Web Map Service](https://www.ogc.org/standards/wms) (WMS) implementation (versions 1.3.0).
Only (tiled) "Basic WMS" is supported at this time.

| Field Name      | Type                 | Description |
| --------------- | -------------------- | ----------- |
| rel             | string               | **REQUIRED**. Must be set to `wms`. |
| href            | string               | **REQUIRED**. Link to the WMS, without any WMS specific query parameters. |
| type            | string               | The media type to be used for the tile requests, e.g. `image/png` or `image/jpeg`. |
| wms:layers      | \[string]            | **REQUIRED**. The layers to show on the map by default. Can't be empty. |
| wms:styles      | \[string]            | The styles to show on the map by default. If not provided or empty, an empty string will be used for the query parameter. |
| wms:dimensions  | Map\<string, string> | Any additional dimension parameters to add to the request as query parameters (e.g. the dimensions `TIME` or `ELEVATION`). |
| wms:transparent | boolean              | Sets whether the layers should be rendered transparent or not. Default: `false` |

If you provide multiple array elements in `wms:layers` (e.g. `["layerA", "layerB"]`),
each should occur in a separate layer in the mapping library so that requests for the layers are sent.
If you want to send multiple layers in a single request, provide them as a string with comma-separated values: `["layerA,layerB"]`.
`wms:layers` and `wms:styles` work in parallel, so the first style in the array will be used for the first layer, etc.

- [More details on the mapping between WMS query parameters and the STAC fields](./wms.md)

### OGC WMTS

Links to a [OGC Web Map Tile Service](https://www.ogc.org/standards/wmts) (WMTS) implementation (versions 1.x).

| Field Name      | Type                 | Description |
| --------------- | -------------------- | ----------- |
| rel             | string               | **REQUIRED**. Must be set to `wmts`. |
| href            | string               | **REQUIRED**. Link to the WMTS, without any WMTS specific parameters. |
| href:servers    | \[string]            | See [href:servers](#hrefservers) below for details. |
| wmts:layer      | string\|\[string]    | **REQUIRED**. The layers to show on the map, either a list of layer names (KVP only) or a single layer name (REST and KVP). |
| wmts:encoding   | string               | The WMTS request encoding, either `kvp` for [KVP](#kvp) or `rest`for [REST](#rest). Defaults to `kvp`. |

**href**:

- For REST encoding, the `href` must point to the capabilities document.
- For KVP encoding, the `href` points to the URL of the capabilities document without any query parameters.
  So if your Capabilities can be requested from `https://example.com/geoserver/service/wmts?service=wmts&request=GetCapabilities`
  you'd provide `https://example.com/geoserver/service/wmts` as `href`.
- The `href` can contain an optional server placeholder `{s}`. If `{s}` is used, the field [`href:servers`](#hrefservers) MUST be provided.

**wmts:layer**: If you provide multiple array elements in `wmts:layer` (e.g. `["layerA", "layerB"]`),
each should occur in a separate layer in the mapping library so that individual requests for the layers are sent.
For REST-encoding only a single string can be provided.

#### KVP

This describes the key-value-pair (KVP) request encoding,
which uses query parameters to transmit additional parameters.

| Field Name      | Type                 | Description |
| --------------- | -------------------- | ----------- |
| type            | string               | The media type to be used for the tile requests, e.g. `image/png` or `image/jpeg`. |
| wmts:encoding   | string               | If provided, must be set to `kvp`. |
| wmts:dimensions | Map\<string, string> | Any additional dimension parameters to add to the request as key-value-pairs (i.e. query parameters). |

#### REST

This describes the REST request encoding, which provides a URL template with variables.

| Field Name    | Type             | Description |
| ------------- | ---------------- | ----------- |
| type          | string           | If provided, must be set to `application/xml`. |
| wmts:encoding | string           | **REQUIRED**. Must be set to `rest`. |
| uriTemplate   | string           | Can override the URL template from the capabilities document with a custom version that can be customized with the `variables` provided. |
| variables     | Map\<string, \*> | This object contains one key per substitution variable in the `urlTemplate`. |

**variables**: This object may provide a JSON Schema definition for any variable defined in the `uriTemplate`,
allowing to provide the default value for the specific STAC resource or the specification of ranges or other constraints.

The variables `{TileMatrixSet}`, `{TileMatrix}`, `{TileRow}`, `{TileCol}` do not need to be provided
explicitly in the `variables` as they can be determined from the capabilities.
Nevertheless, the corresponding variables can be provided if they should be further restricted to certain values or other contraints.

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

| Field Name      | Type                 | Description |
| --------------- | -------------------- | ----------- |
| rel             | string               | **REQUIRED**. Must be set to `pmtiles`. |
| href            | string               | **REQUIRED**. Link to a PMTiles file (usually ends with `.pmtiles`). |
| type            | string               | Recommended to be set to `application/vnd.pmtiles`. |
| pmtiles:layers  | \[string]            | For vector tiles, the layers to show on the map by default. If not provided, it's up to the discretion of the implementation to choose a layer from the `vector_layers` in the PMTiles metadata. |

The [Tile Type](https://github.com/protomaps/PMTiles/blob/main/spec/v3/spec.md#tile-type-tt) of the
PMTiles data source can be read from the first 127 bytes of the the binary header.

It is typical to assume a tile size of 256x256 display (CSS) pixels for raster tiles and 512x512 display pixels for vector tiles.
Tile sizes are not guaranteed to be uniform across an entire archive.

### XYZ

Links to a XYZ, also known as slippy map.

| Field Name   | Type      | Description |
| ------------ | --------- | ----------- |
| rel          | string    | **REQUIRED**. Must be set to `xyz`. |
| href         | string    | **REQUIRED**. Link to the XYZ as a templated URI. |
| type         | string    | Recommended to be set to the image file type the XYZ returns by default, usually `image/png` or `image/jpeg`. |
| href:servers | \[string] | See [href:servers](#hrefservers) below for details. |

**href**: For XYZ, the `href` is a templated URI.
It MUST include the following placeholders: `{x}`, `{y}` and `{z}` and MAY include a placeholder for the server: `{s}`.
If `{s}` is used, the field [`href:servers`](#hrefservers) MUST be provided.
All other parameters should be [hard-coded](https://github.com/stac-extensions/web-map-links/issues/2) with specific values,
e.g. the `{r}` parameter in Leaflet could be replaced by `2x`.

### TileJSON

Links to a [TileJSON](https://github.com/mapbox/tilejson-spec) document.

| Field Name | Type   | Description |
| ---------- | ------ | ----------- |
| rel        | string | **REQUIRED**. Must be set to `tilejson`. |
| href       | string | **REQUIRED**. Link to the valid TileJSON document. |
| type       | string | Recommended to be set to `application/json`. |

### General

The following field applies to multiple types of web mapping services:

| Field Name   | Type      | Description |
| ------------ | --------- | ----------- |
| href:servers | \[string] | A list of replacement values for `{s}` in `href`s. |

#### href:servers

This field is used to specify a set of URLs for a web mapping library so that requests can be sent to multiple servers,
which can avoid request limits in web browsers.

The field is used across multiple types of web mapping services and applies currently to XYZ and WMTS.

It is **REQUIRED** if `{s}` is used in the `href` and then a list of at least 2 allowed values for the placeholder `{s}` must be provided.
If you only have a single value don't provide `href:servers` and instead hard-code the value into the href.

The implementations can expand the given values into multiple URLs.
For example, if you provide `https://{s}/example` as `href` and `href:servers` is `["a.com", "b.eu"]`
you can expand that to `["https://a.com/example", "https://b.eu/example"]`.

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
