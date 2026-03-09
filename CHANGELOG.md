# Changelog

All notable changes to the Miklat MCP Data repository will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [1.0.0] - 2026-03-09

### Added

- **Jerusalem shelter data** — 545 public shelters in GeoJSON format covering neighborhoods across Jerusalem, sourced from the [JLM-Shelters-Dot-Com](https://github.com/danielrosehill/JLM-Shelters-Dot-Com) project.
- **Data manifest** (`data-manifest/manifest.json`) — canonical registry of 199 tracked Israeli municipalities and regional councils with IDs, names (English/Hebrew), district, population, and raw source URLs.
- **Data dictionary** (`data-dictionary/shelter-schema.md`) — full field-level documentation for the shelter GeoJSON schema and area manifest.
- **Three-stage data pipeline** (`pipeline/`) with `first-entry`, `structured`, and `validated` stages for processing raw municipal data into production-ready GeoJSON.
- **Contributing guide** (`CONTRIBUTING.md`) — instructions for submitting shelter data via pull request or direct contact.
- **Pipeline tracking** (`pipeline/first-entry/tracking.json`) — status tracker for each area's progress through the data pipeline.
