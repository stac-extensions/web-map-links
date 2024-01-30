# Changelog
All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- New property `wms:transparent` for WMS
- Roles to specify the layer type, visibility and default layers.

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

[Unreleased]: <https://github.com/stac-extensions/web-map-links/compare/v1.2.0...HEAD>
[1.2.0]: <https://github.com/stac-extensions/web-map-links/compare/v1.1.0...v1.2.0>
[1.1.0]: <https://github.com/stac-extensions/web-map-links/compare/v1.0.0...v1.1.0>
[1.0.0]: <https://github.com/stac-extensions/web-map-links/tree/v1.0.0>
