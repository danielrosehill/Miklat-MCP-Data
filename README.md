# Miklat MCP Data

Community-maintained geodata repository for Israeli public shelters (miklatim tziburim). This repository serves as the upstream data source for the [Miklat MCP](https://github.com/danielrosehill/Miklat-MCP) server.

## Purpose

The Miklat MCP server provides AI agents with tools to find public shelters in Israel via the [Model Context Protocol](https://modelcontextprotocol.io). The shelter geodata powering it lives here, in a separate repository, so that:

- **Community contributions** can be reviewed and merged independently of the MCP server code
- **Data quality** can be validated before it reaches the live MCP endpoint
- **New cities** can be added by contributors who don't need to work with the server codebase

Data from this repository is periodically pulled downstream into the MCP server and deployed.

## Repository Structure

```
data-manifest/
  manifest.json          # Ground truth: all 199 areas, IDs, and raw source URLs
data/
  <area>/shelters.json   # Production-ready GeoJSON (consumed by MCP server)
working-data/
  tracking.json          # Pipeline status tracker for each area
  <area>/                # Work-in-progress data before validation
structured-data/
  <area>/                # Structured intermediate data
  shelter-resources.json # Municipal resource directory
```

Each area (city, local council, or regional council) has a unique identifier (`MKT-001` through `MKT-199`) defined in the [data manifest](data-manifest/manifest.json). The manifest is the single source of truth for what areas are tracked and where their raw data can be found.

## Schema

Each shelter is a GeoJSON Feature with the following structure:

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

### Field Reference

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | integer | Yes | Unique numeric ID within the city dataset |
| `geometry.coordinates` | [lng, lat] | Yes | WGS84 coordinates as `[longitude, latitude]` |
| `slug` | string | Yes | URL-friendly unique identifier (e.g., `shelter-1`) |
| `name` | string | Yes | Display name of the shelter |
| `neighborhood` | string | Yes | Neighborhood or area name |
| `address` | string | Yes | Street address |
| `shelterType` | string | Yes | One of: `Public Shelter`, `Protected Parking`, `Residential Protected Parking`, `School Shelter` |
| `capacity` | integer \| null | No | Number of people the shelter can hold |
| `accessible` | boolean | Yes | Whether the shelter is wheelchair accessible |
| `mapAddress` | string | Yes | Address as used for map lookups |
| `designatedFloors` | string \| null | No | Description of designated safe floors |
| `deepestFloor` | integer \| null | No | Deepest underground floor (negative number, e.g., `-3`) |
| `alertZone` | string \| null | No | Home Front Command alert zone name |

## Contributing

Contributions of shelter data for any Israeli municipality or region are welcome. See **[CONTRIBUTING.md](CONTRIBUTING.md)** for full details including:

- How to submit a pull request with shelter data
- The data schema, data dictionary, and format specification
- How to contribute raw URLs or source data you can't convert yourself
- Contact information for discussing data formats

Municipalities and official data providers are especially encouraged to contribute directly.

## Data Sources

- **Jerusalem**: Sourced from the [JLM-Shelters-Dot-Com](https://github.com/danielrosehill/JLM-Shelters-Dot-Com) project

## License

Shelter data is licensed under the [Open Database License (ODbL)](https://opendatacommons.org/licenses/odbl/1-0/).

## Downstream

This data is consumed by the [Miklat MCP](https://github.com/danielrosehill/Miklat-MCP) server, which exposes it to AI agents via the Model Context Protocol at:

```
https://mcp.jlmshelters.com/mcp
```

## Disclaimer

Shelter data is gathered periodically from official sources and no guarantee is offered as to its accuracy or completeness. Shelters may be added, removed, or changed between updates. Do not rely solely on this data for personal safety or emergency preparedness. Always verify shelter locations with official municipal sources and follow instructions from local authorities during emergencies.
