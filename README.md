# open-apqc — APQC Process Classification Framework catalog

Tranche F scaffolding placeholder. Open spec + catalog data for the APQC PCF
(Process Classification Framework). Consolidates the open-source portion of
the Kyber APQC/BPMN Projector (ADR-0025) — vendor-side customer mapping
stays in `etzhayyim/etzhayyim-root`.

## Status

Phase 2 (scaffolding) per ADR-2605172400. No content yet — Phase 3 will copy
the catalog data + projector spec from vendor.

## Scope

- APQC PCF reference catalog (13 L1 + L2/L3/L4/L5 process taxonomy)
- BPMN 2.0 task catalog mapping
- PCF → BPMN projection spec
- `com.etzhayyim.apqc.*` lexicons (see `../../orgs/etzhayyim/com-etzhayyim-apqc/lex/`)

## Out of scope (stays vendor)

- Customer-specific PCF → BPMN mappings (each customer's instantiated process catalog)
- RisingWave streaming MV for coverage aggregation
- Tenant-isolated projector deployments

## See also

- [`60-apps/etzhayyim-project-open-kyber/`](../etzhayyim-project-open-kyber) — Tranche E open-source ERP that consumes this catalog
- [`orgs/etzhayyim/com-etzhayyim-apqc/lex/`](../../orgs/etzhayyim/com-etzhayyim-apqc/lex) — Tranche F lexicons
- ADR-2605172400 (vendor: 3-axis split rule + Tranche F)
- [ADR-0025 Kyber APQC/BPMN Projector Consolidation](https://github.com/etzhayyim/etzhayyim-root/blob/main/90-docs/adr/0025-kyber-apqc-bpmn-projector-consolidation.md) (foundational)

## Citation catalog (axis-ingest)

Public sources that make this leaf's PCF + BPMN claims falsifiable live in
`facts/catalog.edn`. Live `apqc.org` returns **403** to automated clients, so
publisher identity is pinned through Wayback Machine snapshots that still answer
200; BPMN mapping targets are fetched from `omg.org` directly.

```bash
nbb tools/verify_citations.cljs --min 13
```

Exit 0 only when every catalog URL returns 2xx (redirects followed) and any
non-empty `:cite/expect-substring` is present in the body. Breaking a URL in the
catalog must make the gate exit 1.
