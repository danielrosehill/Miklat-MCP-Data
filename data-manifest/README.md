# Data Manifest

This directory contains the canonical manifest for all areas (municipalities and regional councils) tracked by the Miklat MCP Data pipeline.

## `manifest.json`

The ground source of truth. Each area entry includes:

| Field | Description |
|-------|-------------|
| `id` | Unique identifier (format: `MKT-001` through `MKT-199`) |
| `english_name` | Area name in English |
| `hebrew_name` | Area name in Hebrew |
| `municipality_type` | `city`, `local_council`, or `regional_council` |
| `district` | Israeli administrative district |
| `population` | Approximate population |
| `folder_name` | Directory name used in `data/` and `pipeline/` subdirectories |
| `raw_urls` | Array of known source URLs with format and notes |

## How this manifest is used

1. **MCP server** pulls production data from `data/<folder_name>/shelters.json`
2. **Contributors** use the manifest to find areas that need data or have raw URLs to scrape
3. **Tracking** (`pipeline/first-entry/tracking.json`) references manifest IDs to track pipeline progress

## Updating the manifest

When adding a new area or discovering a new data source URL, update this manifest first. The manifest is the single point of reference for what areas exist and where their raw data can be found.
