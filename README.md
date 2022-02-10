# Web Map Links Extension Specification

- **Title:** Web Map Links
- **Identifier:** <https://stac-extensions.github.io/web-map-links/v1.0.0/schema.json>
- **Field Name Prefix:** none, but each relation type has potentially a distinct prefix for additional data (e.g. `wmts` and `xyz`)
- **Scope:** Item, Catalog, Collection
- **Extension [Maturity Classification](https://github.com/radiantearth/stac-spec/tree/master/extensions/README.md#extension-maturity):** Proposal
- **Owner**: @m-mohr

This document explains the Web Map Links Extension to the
[SpatioTemporal Asset Catalog](https://github.com/radiantearth/stac-spec) (STAC) specification.
It allows to provide links to web maps for visualization purposes. Currently, OGC WMTS and XYZ are supported.

- Examples:
  - [Item example](examples/item.json): Shows the basic usage of the extension in a STAC Item
  - [Collection example](examples/collection.json): Shows the basic usage of the extension in a STAC Collection
- [JSON Schema](json-schema/schema.json)
- [Changelog](./CHANGELOG.md)

## Link Object Fields

This extension can extend the [Link Object](https://github.com/radiantearth/stac-spec/tree/master/item-spec/item-spec.md#link-object)
used in all STAC entities (Catalogs, Collections, Items). It requires specific relation types to be set for the `rel` field in the
Link Object.

An attribution field is not defined as part of this extension, but it is RECOMMENDED to provide an attribution
in the top-level object of the document via the `attribution` field as defined in
[OGC API - Commons - Part 1](http://docs.ogc.org/DRAFTS/19-072.html#landing-page).

### OGC WMTS

Links to a [OGC Web Map Tile Service](https://www.ogc.org/standards/wmts) (WMTS) implementation (versions 1.x).

| Field Name      | Type                 | Description                                                                                                      |
| --------------- | -------------------- | ---------------------------------------------------------------------------------------------------------------- |
| rel             | string               | **REQUIRED**. Must be set to `wmts`.                                                                             |
| href            | string               | **REQUIRED**. Link to the WMTS, without any WMTS specific query parameters.                                      |
| wmts:layer      | string\|\[string]    | **REQUIRED**. The layers to show on the map, either a list of layer names or a single layer name.                |
| wmts:dimensions | Map\<string, string> | Any additional dimension parameters to add to the request as key-value-pairs, usually added as query parameters. |

### XYZ

Links to a XYZ, also known as slippy map.

| Field Name  | Type   | Description                                                                                                                                                        |
| ----------- | ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| rel         | string | **REQUIRED**. Must be set to `xyz`.                                                                                                                                |
| href        | string | **REQUIRED**. Link to the XYZ as a templates URI. MUST include the following placeholders: `{x}`, `{y}` and `{z}`. MAY include a placeholder for the server: `{s}` |
| xyz:servers | array  | REQUIRED if `{s}` is used in the `href`. A list of allowed values for the placeholder `{s}`.                                                                       |

## Asset Object Fields

In addition to the above link extensions,
[Asset Objects](https://github.com/radiantearth/stac-spec/tree/master/item-spec/item-spec.md#asset-object)
can specify a specific role that links to a [TileJSON](https://github.com/mapbox/tilejson-spec) document.

| Field Name | Type       | Description                                        |
| ---------- | ---------- | -------------------------------------------------- |
| roles      | `[string]` | **REQUIRED**. Must include `tiles` as a role.      |
| href       | ``string   | **REQUIRED**. Link to the valid TileJSON document. |
| type       | `string`   | RECOMMENDED to be set to `application/json`        |

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
