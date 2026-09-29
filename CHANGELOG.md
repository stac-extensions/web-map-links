# Changelog
All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- Fields `min_resolution` and `max_resolution` for all web map links to indicate in which resolutions a layer provides useful content
- XYZ: Fields `tile_matrix_set`, `min_zoom` and `max_zoom`
- XYZ: Placeholder `{-y}` for rows counted from the bottom (TMS)
- XYZ: Support for vector tiles, incl. the field `layers`
- WMTS: Field `tile_matrix_set` to select the TileMatrixSet from the capabilities
- TileJSON: Field `layers` for vector tiles
- Documentation for the migration from v1

### Changed

- Each web map link represents exactly one map layer
- XYZ must be provided as link template in `linkTemplates` (Link Templates Extension) with `uriTemplate` instead of `href`
- XYZ: The servers for `{s}` are provided as `enum` of the variable `s` instead of `href:servers`
- WMS: `wms:layers` renamed to `layers`, all layers are requested together in a single request
- WMS: `wms:styles` are parallel to `layers` and allow empty strings for the default style
- WMS and WMTS: `wms:dimensions` and `wmts:dimensions` renamed to `dimensions`, also applies to the REST encoding of WMTS
- WMTS: `wmts:layer` renamed to `layers` and requires an array with exactly one element
- WMTS: The `type` in the REST encoding is the media type of the tiles, not `application/xml`
- PMTiles: `pmtiles:layers` renamed to `layers`

### Removed

- XYZ: `href:servers` (use the variable `s` instead)

## [1.3.0]

### Added

- PMTiles
- WMTS: REST request encoding
- New property `wms:transparent` for WMS

### Changed

- The `type` for WMS and WMTS links should be set to the image media type that the client should request for tiles.
- WMS supports only v1.3.0 of the specification

### Deprecated

### Removed

### Fixed

- WMS: Clarified the behavior of `wms:layers` and `wms:styles`

## [1.2.0]

### Added

- PMTiles

## [1.1.0]

### Added

- OGC WMS (Basic WMS only)
- 3D Tiles

### Changed

- The JSON schema for `wmts:layer` is less ambiguous and a bit more strict

## [1.0.0]

- Initial release for TileJSON, WMTS and XYZ.

[Unreleased]: <https://github.com/stac-extensions/web-map-links/compare/v1.3.0...HEAD>
[1.3.0]: <https://github.com/stac-extensions/web-map-links/compare/v1.2.0...v1.3.0>
[1.2.0]: <https://github.com/stac-extensions/web-map-links/compare/v1.1.0...v1.2.0>
[1.1.0]: <https://github.com/stac-extensions/web-map-links/compare/v1.0.0...v1.1.0>
[1.0.0]: <https://github.com/stac-extensions/web-map-links/tree/v1.0.0>
