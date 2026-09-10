# CDIF Data Structure (profile module)

This repository holds the published artifacts for the **CDIF Data Structure profile module** — the `cdifDataStructure` building block from the [metadataBuildingBlocks](https://github.com/Cross-Domain-Interoperability-Framework/metadataBuildingBlocks) source register.

> **Scope.** `cdifDataStructure` carries machine-readable data-structure metadata: keys, components, foreign keys, and the dimensional/long/wide structure variants. It is consumed by the composite application profile [doc-discoverydatadescriptionstructure](https://github.com/Cross-Domain-Interoperability-Framework/doc-discoverydatadescriptionstructure).

## Specification

- **[CDIFDataStructureImplementationGuide.md](CDIFDataStructureImplementationGuide.md)** — Implementation guide (auto-generated draft; hand-curated content pending).
- **[cdifDataStructureStructuredSchema.json](cdifDataStructureStructuredSchema.json)** — Resolved JSON Schema (Draft 2020-12) generated from the source register.
- **[dataStructureRules.shacl](dataStructureRules.shacl)** — Self-contained SHACL shapes, merged from every composing building block plus the profile-level shapes.
- **[CDIFDataStructure-frame.jsonld](CDIFDataStructure-frame.jsonld)** — JSON-LD frame for Dataset-rooted documents (a `schema:Dataset` whose distribution carries the structure via `cdi:isStructuredBy`).
- **[CDIFDataStructure-structure-frame.jsonld](CDIFDataStructure-structure-frame.jsonld)** — JSON-LD frame for **bare** DataStructure documents (root `@type` = `cdi:DataStructure` / `cdi:DimensionalDataStructure` / `cdi:LongDataStructure` / `cdi:WideDataStructure`). `FrameAndValidate.py` auto-selects between the two frames by the document's root `@type`.

## Examples

`examples/` holds JSON-LD examples illustrating the dimensional, long, wide, and minimal-complete data-structure shapes. Validate one with:

```bash
python FrameAndValidate.py examples/exampleCdifDataStructureComplete.json --validate
```

`FrameAndValidate.py` selects the matching frame by the document's root `@type` — the Dataset frame for `schema:Dataset` records, the structure frame for bare `cdi:*DataStructure` documents — array-wraps the multi-valued properties, then validates against the JSON Schema. Validation is open-world: unknown properties pass.

## Synced from metadataBuildingBlocks

These generated artifacts are re-synced when the source register changes:

| file | source command |
|---|---|
| `cdifDataStructureStructuredSchema.json` | `python tools/resolve_schema.py cdifDataStructure -o cdifDataStructureStructuredSchema.json` |
| `dataStructureRules.shacl` | `python tools/validate_shacl.py cdifDataStructure --emit-shapes dataStructureRules.shacl` |

Source profile: `_sources/profiles/cdifProfile/cdifDataStructure/`.

## Working materials

Background and exploratory material from the profile's development is kept under `archive/` for traceability — DDI-CDI samples and discussion notes, sample data files (`archive/ExampleData/`, `archive/LongData/`, `archive/TestCDIMetadata/`, `archive/XrayAbsorbtion/`, `archive/hierarchicalData/`, `archive/exampleMetadata/`), reference documents (`archive/PhysicalDataset2025.xmi`, SDMX glossary, format-description figures), and the earlier psdi/UML drafts. The `Documents/` directory holds the curated reference docs (`Documents/CDIF-DescribingDatasetStructure.md`, `Documents/hdf5Work/`). None of these are part of the release-artifact set.

## Changelog — v1.1.0

Released 2026-09-10 as `v1.1.0`. Content synced from the CDIF
**metadataBuildingBlocks** source; see the
[release](../../releases/tag/v1.1.0) for the tagged snapshot and
`git log v1.1.0` for the per-commit history:

- **Populated from metadataBuildingBlocks** — `*StructuredSchema.json`, merged SHACL,
  JSON-LD frame, examples, and the normative `FrameAndValidate.py` generated from the
  building-block source; `Examples/` renamed to `examples/`.
- **CDIF v1.1** — profile conformance URIs migrated `/1.0` → `/1.1`.
- **License** standardized on CC-BY-4.0.
- **`@id`-reference tightening** — bare `{@id}` reference slots sealed
  (`additionalProperties: false` + `required: ['@id']`); a canonical `objectReference`
  building block introduced as the strict node reference.
- **`prov:used` wrapper reconciliation** — the base `generatedBy.prov:used` accepts
  role-keyed wrappers (`schema:instrument` / `bios:computationalTool` / `prov:reagent`)
  alongside string / `{@id}` / inline `prov:Entity`; profiles pin a wrapper's shape via
  a constraint-only `if/then` (never a narrowed `anyOf`).
- **`skos:notation` → single string** at concept level (consistent with the codelist
  single-notation design).
- **`FrameAndValidate.py`** (normative, drift-checked against
  `Cross-Domain-Interoperability-Framework/validation`) — two-frame root-`@type`
  selection, context-aware `schema:about`, `--conformance` detection, `cdif:`-`@id`
  re-expansion, and (2026-08) reference-collapse on all document types + blank-node
  dedupe + agent `schema:identifier` unwrap, so `@embed:@always`-framed documents
  validate against the tightened schemas.
- **Examples** conformed to the tightened schemas throughout (PrimaryKey →
  `cdi:ComponentPosition`, reference slots → `{@id}`, CVE `hasIntendedDataType` →
  string, `skos:notation` → string, `schema:additionalType` URI → `{@id}`).


## Branches

`main` is the **current release** — GitHub Pages serves it, so the published
URLs always show the newest release. It is protected: changes reach it only by
pull request, which means the merge *is* the release.

New work goes on the **`updates`** branch and is merged to `main` when a release
is cut, then tagged `v1.1.n`. The former `reviewRevision202606` branch is retained
as **`archive202609`**.

## License

This work is licensed under [Creative Commons Attribution 4.0 International (CC BY 4.0)](LICENSE).
