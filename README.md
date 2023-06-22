# Web Map Links Extension Specification

- **Title:** Web Map Links
- **Identifier:** <https://stac-extensions.github.io/web-map-links/v1.0.0/schema.json>
- **Field Name Prefix:** none, but each relation type has potentially a distinct prefix for additional data (e.g. `wmts` and `xyz`)
- **Scope:** Item, Catalog, Collection
- **Extension [Maturity Classification](https://github.com/radiantearth/stac-spec/tree/master/extensions/README.md#extension-maturity):** Proposal
- **Owner**: @m-mohr

This document explains the Web Map Links Extension to the
[SpatioTemporal Asset Catalog](https://github.com/radiantearth/stac-spec) (STAC) specification.
It allows to provide links to web map services for visualization purposes.

The following services are supported:
- [3D Tiles](#3d-tiles)
- [OGC WMTS](#ogc-wmts)
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
in the top-level object of the document via the `attribution` field as defined in
[OGC API - Commons - Part 1](http://docs.ogc.org/DRAFTS/19-072.html#landing-page).

### 3D Tiles

Links to a [3D Tiles](https://docs.ogc.org/cs/18-053r2/18-053r2.html) implementation (versions 1.x).

| Field Name      | Type                 | Description |
| --------------- | -------------------- | ----------- |
| rel             | string               | **REQUIRED**. Must be set to `3d-tiles`. |
| href            | string               | **REQUIRED**. Link to a tileset. |
| type            | string               | Recommended to be set to `application/json`. |

### OGC WMTS

Links to a [OGC Web Map Tile Service](https://www.ogc.org/standards/wmts) (WMTS) implementation (versions 1.x).

| Field Name      | Type                 | Description |
| --------------- | -------------------- | ----------- |
| rel             | string               | **REQUIRED**. Must be set to `wmts`. |
| href            | string               | **REQUIRED**. Link to the WMTS, without any WMTS specific query parameters. |
| type            | string               | Recommended to be set to the media type the Capabilities document, usually `application/xml`. |
| href:servers    | \[string]            | See [href:servers](#hrefservers) below for details. |
| wmts:layer      | string\|\[string]    | **REQUIRED**. The layers to show on the map by default, either a list of layer names or a single layer name. |
| wmts:dimensions | Map\<string, string> | Any additional dimension parameters to add to the request as key-value-pairs, usually added as query parameters. |

#### href

For WMTS, the `href` is pointing to the URL of the Capabilities document, but without the query parameters for the Capabilities request.
So if your Capabilities can be requested from `https://example.com/geoserver/service/wmts?service=wmts&request=GetCapabilities`
you'd provide `https://example.com/geoserver/service/wmts` as `href`.

The `href` can contain an optional server placeholder `{s}`. If `{s}` is used, the field [`href:servers`](#hrefservers) MUST be provided.

### XYZ

Links to a XYZ, also known as slippy map.

| Field Name   | Type      | Description |
| ------------ | --------- | ----------- |
| rel          | string    | **REQUIRED**. Must be set to `xyz`. |
| href         | string    | **REQUIRED**. Link to the XYZ as a templated URI. |
| type         | string    | Recommended to be set to the image file type the XYZ returns by default, usually `image/png` or `image/jpeg`. |
| href:servers | \[string] | See [href:servers](#hrefservers) below for details. |

#### href

For XYZ, the `href` is a templated URI.
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
