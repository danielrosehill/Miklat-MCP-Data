---
name: import-shelter-source
description: Process a reliable external shelter data source (URL, file, or pasted data) into Miklat-MCP-Data's pipeline — conform to the canonical GeoJSON schema, advance through first-entry → structured → validated → data/, update the manifest + tracking, and record the import date. Use when the user says "import this shelter source", "add this dataset", "process this raw data", or hands you a URL/file containing shelter data for an Israeli area.
---

# Import Shelter Source

Brings a new external shelter data source into the Miklat-MCP-Data repo conforming to the schema in `data-dictionary/shelter-schema.md`.

## Inputs

Ask the user only what is missing:
- **Source** — URL, local file path, or pasted content. Format may be CSV, JSON, GeoJSON, KML, PDF, scraped HTML.
- **Area** — English name or `MKT-NNN` ID. Resolve against `data-manifest/manifest.json` to get `folder_name`. If the area is not in the manifest, stop and ask before adding.
- **Attribution** — author/org name + URL of the source. Required. Goes into the changelog and a per-area `SOURCE.md`.
- **License** — confirm the source's license is compatible (MIT, CC-BY, ODbL, public domain, or explicit permission). If unclear, stop and surface the question.

## Pipeline stages

Always preserve the raw source. Never overwrite an earlier stage in place.

### 1. first-entry — `pipeline/first-entry/<folder_name>/`

- Save the raw source verbatim. Filename: `<YYYY-MM-DD>-<source-slug>.<ext>` (e.g. `2026-05-06-dev-symbol-israel-shelters.geojson`).
- Write/update `pipeline/first-entry/<folder_name>/SOURCE.md` with:
  - Source URL
  - Author / organisation
  - License
  - Retrieval date (today, absolute YYYY-MM-DD)
  - Original record count
  - Any caveats (missing fields, "Unknown address" entries, stale data, etc.)
- Update `pipeline/first-entry/tracking.json` for the area: bump `status` to `in_progress` (or `url_identified` if not yet processed), set `last_updated` at the metadata level.

### 2. structured — `pipeline/structured/<folder_name>/shelters.json`

Transform raw → canonical GeoJSON FeatureCollection per `data-dictionary/shelter-schema.md`:

- **Geometry:** `Point`, coordinates `[longitude, latitude]` (lon first — common bug source). Drop features without valid coords.
- **id:** integer, sequential from 1 within this file.
- **slug:** `shelter-<id>`.
- **Required properties:** `slug`, `name`, `neighborhood`, `address`, `shelterType`, `accessible`, `mapAddress`.
- **Optional:** `capacity`, `designatedFloors`, `deepestFloor`, `alertZone` — use `null` (not `0`, not `""`) when unknown.
- **shelterType:** must be one of `Public Shelter`, `Protected Parking`, `Residential Protected Parking`, `School Shelter`. Map upstream values explicitly; do not invent new types.
- **name fallback:** if source has no name, generate `Shelter <id>`.
- **neighborhood fallback:** if source has no neighborhood, attempt to derive from address; if not derivable, use `Unknown` (note this in SOURCE.md as a quality issue).
- **address fallback:** if source says "Unknown address" or empty, set address to `<area english_name>, Israel` and note in SOURCE.md.
- **accessible:** default `false` if source is silent — never guess `true`.

Write a small transform script alongside the data at `pipeline/structured/<folder_name>/transform.<ext>` (Python or TS) so the conversion is reproducible. Commit the script.

### 3. validated — `pipeline/validated/<folder_name>/shelters.json`

Run validation; only promote if all checks pass. Validation checks:

- File parses as valid JSON and as valid GeoJSON FeatureCollection.
- Every feature has `type: Feature`, integer `id` ≥ 1, `geometry.type: Point`.
- Coordinates: longitude in `[34.2, 35.9]`, latitude in `[29.4, 33.4]` (Israel + administered territories envelope). Flag — don't auto-drop — outliers.
- IDs are unique and sequential (1..N, no gaps).
- Slugs match `shelter-<id>`.
- Required properties present and non-empty (except where `null` is allowed).
- `shelterType` is one of the four allowed values.
- No duplicate coordinates within ~5m (likely dupes — list them, ask before deduping).

If checks fail, **do not promote**. Report the failures and stop.

### 4. data — `data/<folder_name>/shelters.json`

Copy validated file into production. This is what the MCP server consumes.

## Manifest + tracking updates

- `data-manifest/manifest.json` — bump top-level `metadata.last_updated` to today. If the area's `raw_urls` is empty, add the source URL with `format` and a short `notes` field.
- `pipeline/first-entry/tracking.json` — set the area's `status` to `complete`, `shelter_count` to the final feature count, `has_raw_url` to `true` if applicable, and bump `metadata.last_updated`.

## Changelog

Append to `CHANGELOG.md` under a new `## [Unreleased]` section (create if missing):

```
### Added
- **<Area> shelter data** — <N> shelters imported on YYYY-MM-DD from [<Source name>](<URL>) (<License>). Credit: <author/org>.
```

If the source updates an existing area rather than adding it, use `### Changed` with a note about what changed and the diff in shelter count.

## Per-area attribution file

Always write/update `data/<folder_name>/SOURCE.md` (separate from the pipeline SOURCE.md — this one ships with production data):

```
# <Area> Shelter Data — Source & Attribution

- **Source:** [<Source name>](<URL>)
- **Author / Organisation:** <name>
- **License:** <license>
- **Imported:** YYYY-MM-DD
- **Record count:** <N>
- **Notes:** <quality caveats, transformations applied>

Thanks to <author> for making this data publicly available.
```

## Defaults & conventions

- Dates: absolute `YYYY-MM-DD`. Never relative.
- Folder names: lowercase kebab-case, must match the manifest's `folder_name`.
- All file writes use UTF-8, LF line endings, trailing newline.
- Don't reformat unrelated files. Don't renumber existing IDs in other areas.
- Don't run the MCP deploy — this skill ends at committing data to the repo.

## When NOT to use this skill

- Editing existing Jerusalem data (use a focused edit, not the import flow).
- Sources whose license is unclear or restrictive — surface the question instead.
- Country-wide sources without per-area breakdown (e.g. raw lat/lon dumps) — those need a separate "split-by-area" step first; flag and stop.

## End-of-import summary

Report back to the user, in <10 lines:
- Area + ID
- Final shelter count + delta vs prior
- Source + license
- Files touched (paths)
- Any quality flags raised during validation
- Whether changelog was bumped
