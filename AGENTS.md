# open-apqc AGENTS.md

Tranche F leaf. See README.md, and `docs/operator-quickstart.md` for what runs today.

## Boundary

- **etzhayyim (here)**: PCF reference catalog + BPMN task catalog + projection spec + open lexicons
- **vendor** (`etzhayyim/etzhayyim-root`): customer-specific mappings, RisingWave projector runtime, tenant deploys

## NSIDs

See `orgs/etzhayyim/com-etzhayyim-apqc/lex/`.

## Dependencies

- AT MST + IPFS substrate (ADR-2605172000) — no RisingWave, no Kysely, no pg imports
- On-chain payment for any paid feature (ADR-2605172100) — no Stripe / PayPal / fiat

## Status

Not scaffolding-only any more: `kotoba/` carries the Phase-1 publisher (13 PCF
v7.4 L1 categories, 28 offline tests) and `facts/catalog.edn` carries 17
verified citations behind a live fetch gate. The vendor PCF catalog port from
`etzhayyim/etzhayyim-root` is still a separate, deferred work item, as are the
L2–L5 layers.

Before trusting a green run here, read `docs/operator-quickstart.md` — the
citation gate's exit 2 means "could not answer", and `kotoba/`'s advertised
36/36 is 28 runnable + 8 blocked behind an npm 11.x install failure.
