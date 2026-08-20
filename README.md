# open-apqc — APQC Process Classification Framework catalog

Open spec + catalog data for the APQC PCF (Process Classification Framework).
Consolidates the open-source portion of the Kyber APQC/BPMN Projector
(ADR-0025) — vendor-side customer mapping stays in `etzhayyim/etzhayyim-root`.

**Start here: [`docs/operator-quickstart.md`](docs/operator-quickstart.md)** —
what runs today, with the output each step actually produced.

## Status

Two surfaces are present and independently runnable:

| Surface | Path | State |
|---|---|---|
| Citation catalog + live fetch gate | `facts/catalog.edn`, `tools/verify_citations.cljs` | ✅ 17 citations, gate verified to fail on a broken URL (2026-08-20) |
| kotoba reference implementation — PCF v7.4 L1 | `kotoba/` | ✅ 13 L1 categories inline; 28 pure-helper tests pass offline |
| Live PDS seed / anchor verify | `kotoba/src/{seed,query,verify}.ts` | ⏳ needs PDS credentials — not exercised |
| L2–L5 layers (~80 / 250 / 700 / 1,000 entries) | — | ⏳ future PRs, need a checked-in catalog |
| Vendor PCF catalog port from `etzhayyim/etzhayyim-root` | — | ⏳ deferred |

Earlier revisions of this file described the repo as "Phase 2 scaffolding, no
content yet". That stopped being true once `kotoba/` landed the Phase-1
publisher and `facts/catalog.edn` landed the citation set; see
`kotoba/README.md` for the per-layer plan.

One caveat worth carrying: `kotoba/README.md` advertises `36/36` tests, but
only 28 of those run without the `@etzhayyim/sdk` install, which is currently
blocked on npm 11.x. The quickstart records the measurement.

## Scope

- APQC PCF reference catalog (13 L1 + L2/L3/L4/L5 process taxonomy)
- BPMN 2.0 task catalog mapping
- PCF → BPMN projection spec
- `com.etzhayyim.apqc.*` lexicons (see `../../orgs/etzhayyim/com-etzhayyim-apqc/lex/`)

## Out of scope (stays vendor)

- Customer-specific PCF → BPMN mappings (each customer's instantiated process catalog)
- RisingWave streaming MV for coverage aggregation
- Tenant-isolated projector deployments

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
catalog must make the gate exit 1 — verified 2026-08-20 by breaking
`:apqc/wayback-home-2024` and confirming the gate named that entry and exited 1.
Exit 2 is reserved for "could not answer" so a run that checked nothing can
never look like a run that found nothing wrong.

## See also

- [`60-apps/etzhayyim-project-open-kyber/`](../etzhayyim-project-open-kyber) — Tranche E open-source ERP that consumes this catalog
- [`orgs/etzhayyim/com-etzhayyim-apqc/lex/`](../../orgs/etzhayyim/com-etzhayyim-apqc/lex) — Tranche F lexicons
- ADR-2605172400 (vendor: 3-axis split rule + Tranche F)
- [ADR-0025 Kyber APQC/BPMN Projector Consolidation](https://github.com/etzhayyim/etzhayyim-root/blob/main/90-docs/adr/0025-kyber-apqc-bpmn-projector-consolidation.md) (foundational)
