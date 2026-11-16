# sg-csa-ccop-cii-oscal

Machine-readable OSCAL 1.1.2 catalogs for the **Cybersecurity Code of Practice
for Critical Information Infrastructure**, issued by the Cyber Security Agency
of Singapore under section 35A of the Cybersecurity Act 2018.

> **Unofficial.** Source content remains © Government of Singapore. This
> repository is a community conversion published for interoperability with
> [OSCAL](https://pages.nist.gov/OSCAL/) tooling. The upstream PDFs are the
> authoritative text. See [`NOTICE`](NOTICE).

## Releases

One catalog per published version of the CCoP. Files are prefixed with the
release slug so multiple versions coexist cleanly in the same consumer.

| Slug | Version | Effective | Source PDF | Catalog (rendered) |
|---|---|---|---|---|
| `2022-r1` | Second Edition, Revision 1 | 2022-12-12 | [CCoP 2022](https://isomer-user-content.by.gov.sg/36/2df750a7-a3bc-4d77-a492-d64f0ff4db5a/CCoP---Second-Edition_Revision-One.pdf) | [ccop-cii-2022-r1.catalog.md](./ccop-cii-2022-r1.catalog.md) · [.json](./ccop-cii-2022-r1.catalog.json) · [.yaml](./ccop-cii-2022-r1.catalog.yaml) · [provenance](./ccop-cii-2022-r1.provenance.json) |
| `2026-r1` | 2026 Release 1 | 2026-07-29 | [CCoP 2026](https://isomer-user-content.by.gov.sg/36/8b555ddb-16ab-486c-9fda-e12197a4e320/Cybersecurity%20Code%20of%20Practice%20(2026).pdf) | [ccop-cii-2026-r1.catalog.md](./ccop-cii-2026-r1.catalog.md) · [.json](./ccop-cii-2026-r1.catalog.json) · [.yaml](./ccop-cii-2026-r1.catalog.yaml) · [provenance](./ccop-cii-2026-r1.provenance.json) |

## Contents (per release)

| File | Purpose |
|---|---|
| `ccop-cii-<slug>.catalog.json` | OSCAL 1.1.2 catalog (canonical) |
| `ccop-cii-<slug>.catalog.yaml` | Same catalog, YAML serialisation |
| `ccop-cii-<slug>.catalog.md`   | Human-readable rendering |
| `ccop-cii-<slug>.provenance.json` | Builder commit SHA, source URL, PDF SHA256, build time |

Plus shared `CHANGELOG.md`, `NOTICE`, `LICENSE`.

## Stats

| Release | Sections | Subsections | Clauses (controls) | Back-matter resources |
|---|---|---|---|---|
| `2022-r1` | 11 | 51 | 178 | 3 (source PDF, Document History, Annex A) |
| `2026-r1` | 12 | 60 | 218 | 4 (source PDF, Document History, Compliance Timeline, Annex A) |

## Validation

Each catalog validates clean against:
- OSCAL 1.1.2 JSON Schema
- [`oscal-cli`](https://github.com/metaschema-framework/oscal-cli) 3.2.0
  (metaschema + Schematron constraints)

## Mapping from PDF to OSCAL

| CCoP element | OSCAL mapping |
|---|---|
| Numbered section (e.g. `3`, `12`) | depth-1 `group` with `class: ccop-section` |
| Numbered subsection (e.g. `3.1`, `12.4`) | nested `group` with `class: ccop-subsection` |
| Section/subsection preamble prose | group `parts[]` with `name: overview` |
| Numbered clause (e.g. `3.1.4`, `12.7.6`) | `control` with `class: ccop-provision` and a `statement` part |
| Document History page | back-matter `resource` (`type=document-history`) |
| Compliance Timeline page | back-matter `resource` (`type=compliance-timeline`) |
| Annex A | back-matter `resource` (`type=annex`) |
| Source PDF | back-matter `resource` with `rlinks` + SHA-256 hash |

## Identifiers

- Deterministic UUIDv5 under per-release namespace
  `uuid5(DNS, "<slug>.ccop-cii.csa.gov.sg")`. Same clause number in two
  releases yields two distinct UUIDs, letting consumers reason about both
  without collision.
- Control IDs mirror CCoP clause numbering: `ccop-3.1.2`, `ccop-12.4.3`, …
- Group IDs: `ccop-g-3`, `ccop-g-3.1`, …
- Each clause control carries `label` (clause number) and `source-page`
  props so downstream renderers can point back to the PDF.

Re-generating from the same inputs produces identical UUIDs — stable across
builds for diffing and cross-reference.

## Build process (summary)

Catalogs are produced by [`sg-csa-ccop-cii-oscal-builder`](https://github.com/badenh/sg-csa-ccop-cii-oscal-builder).

```
fetch      ← upstream PDF (SHA256 pinned per release)
extract    ← pdfplumber walks the TOC + body into a sectioned tree
transform  ← tree → OSCAL 1.1.2 catalog (deterministic UUIDv5)
validate   ← JSON Schema + oscal-cli (metaschema + Schematron)
publish    ← catalog.{json,yaml,md} + provenance.json
```

Builds are deterministic: the same builder commit + source PDF SHA256 produce
byte-for-byte identical OSCAL output. The PDF is **not** redistributed from
this repository; `rlinks` in the catalog point at the canonical CSA URL and
`provenance.json` records the exact SHA256 consumed.

## Source

- Issuer: Cyber Security Agency of Singapore
- Legal basis: section 35A, Cybersecurity Act 2018
- Landing page: https://www.csa.gov.sg/legislation/codes-of-practice/
- Copyright: © Government of Singapore

## Converter

- Author: Baden Hughes
- License: Apache-2.0 (applies to conversion artifacts in this repository)
- Builder: these artefacts are machine-generated, via a private project
  `sg-csa-ccop-cii-oscal-builder`.

## Using the catalogs

```python
import httpx

RAW = "https://raw.githubusercontent.com/badenh/sg-csa-ccop-cii-oscal/main"

for slug in ("2022-r1", "2026-r1"):
    cat = httpx.get(f"{RAW}/ccop-cii-{slug}.catalog.json").json()["catalog"]
    print(slug, cat["metadata"]["title"], cat["metadata"]["version"])
```

Or from an OSCAL profile that imports a specific release by URL:

```yaml
profile:
  imports:
    - href: "https://raw.githubusercontent.com/badenh/sg-csa-ccop-cii-oscal/main/ccop-cii-2026-r1.catalog.json"
      include-all: {}
```

## Reporting issues

These artefacts are machine-generated, and the upstream sources themselves
may contain errors or inconsistencies. If you spot a problem or have a
question, please raise an issue in the GitHub project — the maintainer will
address it in the upstream extractor / generator / validator and flow the
fix back down into updated releases.

Questions about the source text itself should go to the Cyber Security Agency
of Singapore.
