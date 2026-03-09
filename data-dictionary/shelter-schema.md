# Shelter Data Dictionary

This document describes every field in the production shelter GeoJSON files (`data/<area>/shelters.json`).

## File Format

- **Format:** GeoJSON FeatureCollection ([RFC 7946](https://tools.ietf.org/html/rfc7946))
- **Encoding:** UTF-8
- **Coordinate Reference System:** WGS84 (EPSG:4326)
- **File name:** `shelters.json` (one per area)

## Top-Level Structure

| Field | Type | Description |
|-------|------|-------------|
| `type` | string | Always `"FeatureCollection"` |
| `features` | array | Array of Feature objects (one per shelter) |

## Feature Object

| Field | Type | Description |
|-------|------|-------------|
| `type` | string | Always `"Feature"` |
| `id` | integer | Unique numeric ID within this area's dataset. Starts at 1 and increments sequentially. |
| `geometry` | object | GeoJSON Point geometry (see below) |
| `properties` | object | Shelter metadata (see below) |

## Geometry

| Field | Type | Description |
|-------|------|-------------|
| `type` | string | Always `"Point"` |
| `coordinates` | [number, number] | `[longitude, latitude]` in WGS84. Must be accurate to at least 5 decimal places. Longitude first per GeoJSON spec. |

## Properties

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `slug` | string | Yes | URL-friendly unique identifier within this dataset. Format: `shelter-<id>` (e.g., `shelter-1`, `shelter-42`). |
| `name` | string | Yes | Human-readable display name for the shelter (e.g., `"Shelter 681"`). |
| `neighborhood` | string | Yes | Neighborhood, quarter, or sub-area where the shelter is located (e.g., `"Baka"`, `"Rehavia"`). |
| `address` | string | Yes | Full street address including city and country (e.g., `"8 Efrayim St, Jerusalem, Israel"`). |
| `shelterType` | string | Yes | Classification of the shelter. Must be one of the values listed below. |
| `capacity` | integer or null | No | Maximum number of people the shelter can hold. `null` if unknown. |
| `accessible` | boolean | Yes | `true` if the shelter is wheelchair accessible, `false` otherwise. |
| `mapAddress` | string | Yes | Address string optimized for geocoding and map lookups. Often identical to `address`. |
| `designatedFloors` | string or null | No | Description of which floors are designated as shelter space (e.g., `"Ground floor and basement"`). `null` if not applicable or unknown. |
| `deepestFloor` | integer or null | No | Deepest underground floor level as a negative integer (e.g., `-3` for three floors underground). `null` if not applicable or unknown. |
| `alertZone` | string or null | No | Home Front Command (Pikud HaOref) alert zone name (e.g., `"Jerusalem South"`, `"Dan South"`). `null` if unknown. |

## Shelter Types

| Value | Description |
|-------|-------------|
| `Public Shelter` | A publicly accessible bomb shelter (miklat tziburi) open to the general public during emergencies. |
| `Protected Parking` | An underground parking structure designated as a shelter during emergencies. |
| `Residential Protected Parking` | A residential building's parking area that doubles as a shelter. |
| `School Shelter` | A shelter located within or adjacent to a school compound. |

## Example Feature

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

## Area Manifest Fields

The area-level metadata lives in `data-manifest/manifest.json`. Each area entry contains:

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique area identifier (format: `MKT-001` through `MKT-199`). |
| `english_name` | string | Area name in English. |
| `hebrew_name` | string | Area name in Hebrew. |
| `municipality_type` | string | One of: `city`, `local_council`, `regional_council`. |
| `district` | string | Israeli administrative district (e.g., `"Northern"`, `"Jerusalem"`, `"Southern"`). |
| `population` | integer | Approximate population of the area. |
| `folder_name` | string | Directory name used under `data/` and `pipeline/` (lowercase kebab-case). |
| `raw_urls` | array | Known source URLs for raw shelter data, each with `url`, `format`, and `notes` fields. |
