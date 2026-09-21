# CLAUDE.md — Magyar-Alapellatas (Praxistérkép)

Interactive public map of Hungarian primary care districts (dental first, GP later),
showing vacancies, duration, affected population and trends, built from monthly
NEAK open data. Visual/UX reference: https://holadelej.hu

Full plan: `docs/PLAN.md` (Hungarian). Read it before non-trivial work.

## Non-negotiable rules

1. **Authorship: Dul Zoltán only.** NEVER add `Co-Authored-By: Claude`,
   "Generated with Claude", or any AI attribution to commits, files, docs,
   READMEs, or metadata. No exceptions.
2. **No hallucinated data.** Every number must trace to a source file in
   `data/raw/YYYY-MM/`. If a source is missing or a parser fails validation,
   the pipeline must fail loudly — never publish partial or guessed data.
3. **Public data only; names only where NEAK itself publishes them.** The
   contracted physician's name of a FILLED praxis (from the NEAK public
   registry) may be shown. Names on vacant/dissolved districts and
   substitute physicians — never. Vacancy markers (BETÖLTETLEN) are not
   names. (Policy widened 2026-08 at the owner's request.)
4. **Betöltetlen ≠ ellátatlan.** Keep "vacant" (no contracted physician) and
   "unserved" (no care available incl. substitution) strictly separate in code,
   data model and UI copy.

## Project phase

**MVP (dental) and the v1 GP branch are both implemented.** Every pipeline
component is dual-cassette (`kind: 'dental' | 'gp'`). Note: NEAK publishes no
dissolved (megszűnt) list for GP; the GP registry is the source of district
(járás) names and served-settlement lists.

## Language conventions

- Code, comments, commit messages, identifiers: **English**.
- UI copy, docs for stakeholders, PLAN.md: **Hungarian**. UI strings live in
  `web/src/i18n/hu.json` only — never hardcode Hungarian text in components.
- Hungarian quotes in JS/JSON string literals: use Unicode escapes
  (`\u201E` „ and `\u201D` ”) to avoid encoding issues (known past pain point).

## Architecture

- **No backend.** Static SPA + monthly batch ETL via GitHub Actions.
- ETL: Python 3.12 in `etl/` — pdfplumber, pandas, openpyxl.
  Install with `pip install -r etl/requirements.txt`.
- Data: versioned JSON snapshots in `data/YYYY-MM/`, `data/latest.json`,
  `data/timeseries.json`. Raw source files archived in `data/raw/YYYY-MM/`
  (git-tracked — the git history IS the audit trail).
- Supplements (optional, never block a release): `data/eeszt.json` (EESZT
  match per district), `data/dental_extra.json` (the dental services that
  are NOT districts: on-call, university primary care, every Szakellátás
  service), `data/vedono.json` (the health-visitor branch, EESZT-only — no
  vacancy list exists for it anywhere) and `data/specialist.json` (source I).
- Analyses built on top of the above, each with its own guard:
  `risk.json` (vacancy risk), `survival.json` (Kaplan–Meier of vacancy
  spells), `coverage.json` (settlement-level coverage instead of district
  seats), `composite.json` (the composite care-risk index) and `age.json`
  (KSH census age composition). `data/geo/settlements.geojson` carries a
  centre point per settlement.
- Pages: `index.html` (landing, with the health-visitor view behind its own
  icon), `elemzo.html` (the four analyses), `eeszt.html` (the EESZT
  complex), `szakellato.html` (specialist care), `embed.html`.
- Frontend: React 18 + Vite + TypeScript in `web/`, Zustand for state,
  MapLibre GL JS for the map, D3 for charts. No Tailwind; hand-rolled CSS
  matching the dark holadelej-style theme.
- Geocoding: Nominatim, 1 req/s, results cached in `etl/geocode_cache.json`
  (git-tracked). Only query addresses not in cache.

## Data sources (canonical list — do not invent others)

| ID | Source | Format | Cadence |
|----|--------|--------|---------|
| A | NEAK Betöltetlen fogorvosi szolgálatok | PDF | monthly |
| B | NEAK Betöltetlen háziorvosi szolgálatok | PDF | monthly (v1) |
| C | NEAK szerződött szolgáltatók (registry = denominator) | XLS/PDF | monthly |
| D | NEAK havi finanszírozási adatok (performance) | XLS | monthly (v1) |
| E | OKFŐ tartósan betöltetlen körzetek | HTML/PDF | cross-check |
| F | KSH settlement population (Helységnévtár) | XLS | yearly |
| F2 | KSH 2022 census age composition (settlement level, CC BY 4.0) | REST JSON | one-off |
| G | Settlement/district/county boundaries (OSM/KSH) | GeoJSON | static |
| H | EESZT törzspublikáció (NEAK_FINSZOLG, EUSZOLG_PUBLIKUS, EUSZOLG_ENGEDELY_PUBLIKUS) | REST JSON | supplement (monthly) |
| I | NEAK fekvő- és járóbeteg-szakellátó intézmények (XLSX only — the outpatient PDF is truncated) | XLSX | monthly |

Source URLs are configured in `etl/sources.py` — update there only.
NEAK data is "tájékoztató jellegű"; this disclaimer must appear in the UI footer.

## Key data model

The district denominator is the registry filtered to `Ellátási szint =
Alapellátás` AND a district service type (Felnőtt/Gyermek/Vegyes/Iskolai) for
dental, and to `ellátási forma = T` for GP. Everything else the registry lists
(Ügyelet, Egyetemi alapellátás, Szakellátás) is NOT a district: it never
enters the vacancy rate and lives in `data/dental_extra.json` instead.

`Praxis.id` = NEAK FIN code (canonical key). Praxis types:
`adult | child | mixed | school`. Status:
`filled | vacant | vacant_longterm | dissolved`. A FIN code disappearing from
the registry means `dissolved`, never silently dropped or counted as filled.

## ETL invariants (validate.py must enforce)

- Row count within ±15% of previous month, else fail with diff report.
- Every vacant praxis has: FIN code, county, ≥1 site address, vacantSince.
- Aggregates: settlement counts sum to district, district to county,
  county to national. Any mismatch = hard fail.
- Failed geocode → settlement-centroid fallback, flagged `geoApprox: true`.

## Testing

- ETL: pytest with fixture PDFs in `etl/tests/fixtures/` (commit one real
  anonym-safe sample per source format version).
- Web: Vitest for logic (aggregation selectors, formatting); pure logic
  separated from presentation (same pattern as the arcade game projects).

## Workflow

- Monthly pipeline: `.github/workflows/monthly-etl.yml` (cron, runs twice
  monthly). On validation failure it opens an issue instead of deploying.
- Local run: `python etl/run.py --month 2026-08` then `cd web && npm run dev`.
- Deploy: static build to Cloudflare Pages from `main`.
