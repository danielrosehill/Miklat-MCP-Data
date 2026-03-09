# Contributing to Miklat MCP Data

Thank you for your interest in contributing shelter (miklat) data to this project. This repository is the upstream data source for the [Miklat MCP](https://github.com/danielrosehill/Miklat-MCP) server, which provides AI agents with tools to find public shelters in Israel via the [Model Context Protocol](https://modelcontextprotocol.io).

**MCP endpoint:** `https://mcp.jlmshelters.com/mcp`

## Contact

For questions about data formats, contribution workflows, or to discuss adding data for your municipality:

**Daniel Rosehill** — public@danielrosehill.com

Municipalities and official data providers are especially welcome to reach out directly.

## Use Cases and Deployment

This data powers an MCP server that enables AI assistants to:

- **Find nearby shelters** given a location or address in Israel
- **List shelters by city or neighborhood** with capacity, accessibility, and type information
- **Provide emergency shelter guidance** during security situations

The data flows through this pipeline:

```
Raw municipal data → This repository (validated GeoJSON) → Miklat MCP server → AI agents
```

Any AI assistant that supports MCP can connect to the live endpoint and query shelter data in real time.

## Repository Structure

```
data-manifest/
  manifest.json          # Ground truth: all areas, IDs, and raw source URLs
data/
  <area>/shelters.json   # Production-ready GeoJSON (consumed by MCP)
working-data/
  tracking.json          # Pipeline status for each area
  <area>/                # Work-in-progress data before validation
structured-data/
  <area>/                # Structured intermediate data
  shelter-resources.json # Municipal resource directory
```

## Data Schema

Each shelter is a GeoJSON Feature within a FeatureCollection. The file must be valid [GeoJSON](https://geojson.org/).

### Example Feature

```json
{
  "type": "Feature",
  "id": 1,
  "geometry": {
    "type": "Point",
    "coordinates": [35.2189683, 31.75959347]
  },
  "properties": {
    "slug": "shelter-1",
    "name": "Shelter 681",
    "neighborhood": "Baka",
    "address": "8 Efrayim St, Jerusalem, Israel",
    "shelterType": "Public Shelter",
    "capacity": 80,
    "accessible": false,
    "mapAddress": "8 Efrayim St, Jerusalem, Israel",
    "designatedFloors": null,
    "deepestFloor": null,
    "alertZone": "Jerusalem South"
  }
}
```

### Data Dictionary

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | integer | Yes | Unique numeric ID within the city dataset |
| `geometry.coordinates` | [lng, lat] | Yes | WGS84 coordinates as `[longitude, latitude]`. Must be accurate to at least 5 decimal places. |
| `slug` | string | Yes | URL-friendly unique identifier (format: `shelter-<id>`) |
| `name` | string | Yes | Display name of the shelter |
| `neighborhood` | string | Yes | Neighborhood or area name |
| `address` | string | Yes | Street address including city (e.g., "8 Efrayim St, Jerusalem, Israel") |
| `shelterType` | string | Yes | One of: `Public Shelter`, `Protected Parking`, `Residential Protected Parking`, `School Shelter` |
| `capacity` | integer \| null | No | Number of people the shelter can hold |
| `accessible` | boolean | Yes | Whether the shelter is wheelchair accessible |
| `mapAddress` | string | Yes | Address as used for map lookups |
| `designatedFloors` | string \| null | No | Description of designated safe floors |
| `deepestFloor` | integer \| null | No | Deepest underground floor (negative number, e.g., `-3`) |
| `alertZone` | string \| null | No | Home Front Command alert zone name |

### File Format

- **Format:** GeoJSON FeatureCollection
- **Encoding:** UTF-8
- **Coordinate system:** WGS84 (EPSG:4326)
- **File name:** `shelters.json`
- **One file per area** in its corresponding directory under `data/`

## How to Contribute

### Option 1: Pull Request (preferred)

#### Adding shelters to an existing area

1. Fork this repository
2. Find the area's identifier in `data-manifest/manifest.json` (look up the `folder_name`)
3. Edit `data/<folder_name>/shelters.json`
4. Ensure new features follow the schema above
5. Assign unique `id` and `slug` values that don't conflict with existing entries
6. Submit a pull request with:
   - The area name and its manifest ID (e.g., `MKT-025`)
   - A description of the data source
   - Number of shelters added

#### Adding a new area

1. Fork this repository
2. Create `data/<area-name>/shelters.json` with a valid GeoJSON FeatureCollection
3. Use lowercase kebab-case for the directory name (e.g., `tel-aviv-yafo`)
4. Follow the schema above
5. Add the area to `data-manifest/manifest.json` if it isn't already listed
6. Submit a pull request with:
   - Area name (English and Hebrew)
   - Municipality type (city, local_council, or regional_council)
   - Data source description

#### Submitting raw data or source URLs

If you have a link to a municipal shelter page or PDF but can't convert it to GeoJSON yourself:

1. Fork this repository
2. Add the URL to the relevant area in `data-manifest/manifest.json` under `raw_urls`
3. Submit a pull request describing what the URL contains

### Option 2: Direct Contact

If you prefer not to use GitHub, or if you represent a municipality and want to discuss data sharing:

- Email Daniel at **public@danielrosehill.com**
- Include the area name, data format, and either the data itself or a link to it

We accept data in any format (PDF, spreadsheet, web page, API) and will handle conversion to GeoJSON.

## Data Quality Guidelines

- Coordinates must be accurate to at least 5 decimal places
- Addresses should include the city name (e.g., "8 Efrayim St, Jerusalem, Israel")
- Shelter types must use one of the four standard values listed above
- IDs must be unique integers within each city file
- Slugs must be unique strings within each city file (format: `shelter-<id>`)
- All text fields should be in English (Hebrew names go in a separate field if needed)

## Current Status

Check `working-data/tracking.json` for the current pipeline status of each area. The tracking file shows:

- Which areas have complete production data
- Which areas have identified source URLs but need scraping/conversion
- Which areas still need data sources to be found

## License

By contributing, you agree that your data contributions will be licensed under the [Open Database License (ODbL) 1.0](https://opendatacommons.org/licenses/odbl/1-0/).
