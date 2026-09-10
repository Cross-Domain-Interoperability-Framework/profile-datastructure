# AGENTS.md — AI Agent Guidance for CDIF Data Structure (profile module)

## Project context

This repository publishes the **CDIF Data Structure profile module** (`cdifDataStructure`). It carries the data-structure metadata used by the composite `doc-discoverydatadescriptionstructure` application profile. Module scope: keys, components, foreign keys, and the dimensional / long / wide structure variants.

## Key files

- `CDIFDataStructureImplementationGuide.md` — implementation guide (auto-generated draft from the StructuredSchema; hand-curated content pending)
- `cdifDataStructureStructuredSchema.json` — JSON Schema (generated)
- `dataStructureRules.shacl` — merged SHACL shapes (generated)
- `CDIFDataStructure-frame.jsonld` — JSON-LD frame for Dataset-rooted documents (structure nested under a `schema:Dataset` distribution via `cdi:isStructuredBy`)
- `CDIFDataStructure-structure-frame.jsonld` — JSON-LD frame for **bare** DataStructure documents (root `@type` = `cdi:*DataStructure`); `FrameAndValidate.py` auto-selects by root `@type`
- `examples/` — validated JSON-LD examples: bare-structure (dimensional, long, wide, minimal, complete) under the structure frame; `FeXAS/` Dataset-rooted examples under the Dataset frame
- `FrameAndValidate.py` — frame + JSON Schema validation (multi-frame selection by root `@type`)

## Synced files (manual sync from metadataBuildingBlocks)

These are generated from the source register and must be re-synced when the source changes:

- `cdifDataStructureStructuredSchema.json` ← `python tools/resolve_schema.py cdifDataStructure -o <file>`
- `dataStructureRules.shacl` ← `python tools/validate_shacl.py cdifDataStructure --emit-shapes <file>`

Source profile dir: `metadataBuildingBlocks/_sources/profiles/cdifProfile/cdifDataStructure/`.

## Working materials (non-release)

Working scratch material is kept under `archive/` for traceability:

- DDI-CDI references — `archive/DDI-CDI_SimpleSample.zip`, `archive/DDI-CDI-forDataIntegration.txt`, `archive/DataTypesDiscussion.txt`
- Sample data — `archive/ExampleData/`, `archive/LongData/`, `archive/TestCDIMetadata/`, `archive/XrayAbsorbtion/`, `archive/hierarchicalData/`, `archive/exampleMetadata/`
- Reference material — `archive/PhysicalDataset2025.xmi`, `archive/SDMX_Glossary_*.docx`, format-description figures, the earlier psdi/UML drafts

Curated reference docs live under `Documents/` (`Documents/CDIF-DescribingDatasetStructure.md`, `Documents/hdf5Work/`).

Do not treat any of this as part of the release artifact set; the canonical release files are the `*StructuredSchema.json`, `*Rules.shacl`, `*-frame.jsonld`, `FrameAndValidate.py`, and `examples/`.

## Example conventions

1. `@context` declares explicit prefixes (`schema`, `cdi`, `cdif`, `dcterms`, `dcat`) — never `@vocab`.
2. `@type` as arrays (e.g. `["schema:Dataset"]`); dual-typed where needed (e.g. `["schema:Dataset", "cdi:DimensionalDataStructure"]`).
3. `cdif:physicalDataType` is single-valued everywhere (not array-valued).
4. Never strip unknown properties — validation is open-world.

## Validation

```bash
# schema + frame auto-detected; the frame is chosen by the document's root @type
# (structure frame for bare cdi:*DataStructure docs, Dataset frame for FeXAS/ records)
python FrameAndValidate.py examples/<file>.json --validate
```

Pass `--frame` explicitly only to override auto-selection.

## Development branch

Active development for the 2026-06 review revision targets the `updates` branch; merged to `main` on release.
