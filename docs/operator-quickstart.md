# Operator quickstart — org-apqc-pcf

What you can run in this repo today, with the result each step actually
produced. Every command below was executed on 2026-08-20; where a step does
**not** work, that is recorded as measured rather than omitted.

Measurement host: macOS (darwin 25.3.0), `node v26.3.0`, `npm 11.16.0`.
Version numbers matter here — step 3 fails for a reason that is specific to
the npm major, not to this repo.

## What this repo is

Two surfaces, both real, both independently runnable:

| Surface | Path | Needs network | Needs install |
|---|---|---|---|
| Citation catalog + live gate | `facts/catalog.edn`, `tools/verify_citations.cljs` | yes | no |
| kotoba reference implementation (PCF v7.4 L1) | `kotoba/` | no (tests) | yes |

The citation gate is the one to run first: it needs no package install and it
answers in about a minute.

## 1. Verify the citation catalog

```bash
nbb tools/verify_citations.cljs --min 13
```

Every `:cite/url` in `facts/catalog.edn` is fetched; a row passes when it
answers 2xx (redirects followed) and — if `:cite/expect-substring` is
non-empty — that substring appears in the body.

Measured 2026-08-20:

```
CHECKED 17 OK 17 FAIL 0 MIN 13
PASS
```

Exit codes are three-valued on purpose, and the distinction is the point:

| Exit | Meaning |
|---:|---|
| 0 | answered; every citation checked; floor met |
| 1 | answered; at least one citation is wrong |
| 2 | **could not answer** — parse failure, network, zero checks, or floor miss |

`2` exists so that "nothing was checked" can never be mistaken for "nothing was
wrong". If you are wiring this into anything automated, treat 2 as a hard stop,
not as a pass.

`--min 13` is an evidence floor: fewer than 13 rows checked exits 2 even if all
of them passed. Raise it when the catalog grows; never lower it to make a run
go green.

## 2. Confirm the gate is not theatre

A gate that cannot fail tells you nothing. Verify it discriminates before you
trust a green run — on a scratch copy, never on your working tree:

```bash
cp -R . /tmp/apqc-gate-probe && cd /tmp/apqc-gate-probe && rm -f .git
# point any one :cite/url at a URL that will 404, then:
nbb tools/verify_citations.cljs --min 13; echo "EXIT=$?"
```

Measured 2026-08-20, after breaking `:apqc/wayback-home-2024`:

```
CHECKED 17 OK 16 FAIL 1 MIN 13
FAIL :apqc/wayback-home-2024 HTTP 404
EXIT=1
```

Check that the entry named in the output is the entry you broke. A gate that
goes red for a *different* reason than the one you introduced has not been
demonstrated — it has only been disturbed.

## 3. The kotoba package — what runs and what does not

### `npm install` currently fails on npm 11.x

```bash
cd kotoba && npm install
```

Measured 2026-08-20 on `npm 11.16.0`:

```
npm error code EALLOWSCRIPTS
npm error --allow-scripts is not allowed in project-scoped installs.
npm error git dep preparation failed
```

The cause is the single git dependency, `@etzhayyim/sdk`. npm 11.16 requires
install scripts to be allow-listed, and the *inner* install that npm spawns to
prepare a git dependency does not inherit an `allowScripts` entry or an
`.npmrc` from this project — both were tried on 2026-08-20 and neither helps.
`--ignore-scripts` does not help either; the inner install still runs.

This is not a claim that the package is broken everywhere. Other machines in
this fleet run different npm majors, and the npm-version-dependence of local
tooling has bitten this workspace before. Re-measure before concluding.

The registry dependencies are fine in isolation — `@types/node`, `tsx`,
`typescript` and `vitest` install in about 13 seconds with no git dependency in
the tree. The blocker is exactly one package.

### The pure-helper suite runs offline

`src/types.test.ts` imports only `./types.js` and needs neither the SDK nor the
network. Measured 2026-08-20 with dev dependencies alone:

```
Test Files  1 passed (1)
     Tests  28 passed (28)
```

Those 28 cases lock down the L1 code format: all 13 valid v7.4 codes
(`1.0` … `13.0`) accepted, 11 near-miss variants rejected (`0.0`, `14.0`,
`1.1`, `1`, `1.0.0`, whitespace-padded forms), plus `l1Ordinal` numeric
extraction and its NaN-safe rejection path.

### The seed suite does not run offline

`src/seed.test.ts` documents itself as "No SDK / network", but it imports
`./seed.js`, and `seed.ts` imports `@etzhayyim/sdk` at module top level and
constructs a client there. Measured 2026-08-20:

```
FAIL src/seed.test.ts
Error: Cannot find package '@etzhayyim/sdk' imported from .../src/seed.ts
```

So the suite is SDK-coupled through its import chain even though none of its
assertions touch the network. `kotoba/README.md` advertises `36/36`; on this
host **28 are runnable and 8 are not**. Treat `36/36` as unverified until the
install path or the import chain is fixed.

### Commands that need live credentials

`src/seed.ts`, `src/query.ts` and `src/verify.ts` all talk to a PDS and to Base
L2. They need `ETZ_PDS_URL` plus an authenticated SDK session, which this repo
does not carry. They were **not** exercised here, and this document does not
report a result for them.

## Known gaps (measured 2026-08-20)

- **`npm install` blocked on npm 11.16.** See step 3. Nothing in this repo
  needs to change for the pure-helper suite to pass; the coupling in
  `seed.test.ts` is what keeps 8 cases behind the install.
- **The SDK dependency URL points at a moved repo.** `package.json` pins
  `git+https://github.com/etzhayyim/com-etzhayyim-sdk.git`; that path now
  redirects to `kotoba-lang/sdk` (public). The redirect resolves today, so this
  is latent rather than broken — but a pinned URL that only works via redirect
  is worth re-pinning deliberately rather than discovering later.
- **`kotoba/README.md` relative links are pre-extraction.** They resolve
  against the old `60-apps/etzhayyim-project-open-apqc/` monorepo layout
  (`../../../90-docs/adr/…`, `../../etzhayyim-project-open-isco/`) and escape
  this repository. The correct targets live in other west projects; they are
  left unrewritten here rather than guessed.
