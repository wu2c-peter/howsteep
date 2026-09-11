# HowSteep

**[howsteep.org](https://howsteep.org)** — reproducible LiDAR gradient
ladders for road climbs, so far in the UK, the north-eastern United States,
Hawaii, Switzerland and Austria: wherever open 1 m (or finer) LiDAR and an
open road inventory exist, the survey can go. Austria is measured without
downloading a terrain model at all: the federal 1 m grid is read block by
block from its server, one kilometre at a time. Every maximum states its measurement distance, because a hill
can honestly be 32% over 2 m and 25% over 100 m at once.

This repository is the public face of the project: **corrections,
local knowledge and photo reports** are collected here as issues.
The measurement pipeline lives in a private repository.

## Reporting something

Use the [climb report form](../../issues/new/choose). The most valuable
reports are the ones only a person on the ground can make:

- a gradient **sign** we don't show, or one that has been repainted
  (signs change — we date every observation);
- a road that is **gated, private or unsurfaced** despite our checks;
- a **name** locals actually use for a climb we auto-named;
- a **climb we're missing** entirely;
- **photos** of any of the above — drag them straight into the form.

Discovered climbs on the site are hypotheses derived from LiDAR terrain
data and open road datasets. Despite extensive computational
verification, some can still be artifacts of the terrain model or the
map — that's exactly what your reports help fix.

**Why isn't my hill ranked?** Every road the survey has ever considered is
in the [decisions dataset](https://howsteep.org/data/decisions/): published
and ranked, published but held for a person's look, withdrawn, merged,
measured but never given a page, or proposed and never measured — each with
a coded reason and the rule behind it. Look your road up there first; if the
rule got it wrong, that is exactly what the report form is for. The codes are
explained in [DECISIONS.md](DECISIONS.md).

## The OCL format

The site's measurements are published in **OpenClimb Ladder (OCL)** — a
data format in which a maximum gradient *cannot be expressed without its
measurement distance*. The schema forbids windowless maxima; that is the
point. Spec: [spec/OCL.md](spec/OCL.md) · normative JSON Schema:
[schema/ocl-climb.schema.json](schema/ocl-climb.schema.json) · live
example: any climb page on [howsteep.org](https://howsteep.org) links its
own OCL document. Comments and implementations welcome — open an issue.

Format changelog (all changes within 0.1.x are additive):

- **2026-09-07** — spec and schema brought level with the live corpus:
  `provenance.run_id` (opaque batch-run identity, spec section added
  2026-08-22); per-window lateral-stability fields on ladder entries —
  `placement_sensitivity_pp`, `estimated_best_pct`,
  `estimated_range_pct`, `estimate_basis` — recording how much a maximum
  moves when the measurement line is shifted across the carriageway and
  the resulting estimate of the true reading. Documents published since
  30 August carry them.

## Data & licences

Measurements are derived from open data: national LiDAR programmes of
all four UK nations (OGL v3), OS Terrain 50 / Open Names / Open Roads
(OGL v3), USGS 3DEP LiDAR (public domain) with the states'
road inventories, and OpenStreetMap (ODbL). What the survey publishes,
you may reuse — the terms are settled:

- **any individual climb's page, charts and figures** — CC BY 4.0,
  with credit to "HowSteep — howsteep.org";
- **the complete dataset** (CSV and per-climb OCL documents,
  downloadable from [howsteep.org](https://howsteep.org)) — ODbL 1.0
  (attribution + share-alike, as inherited from OpenStreetMap);
- **the decisions dataset** (every route ever considered, with its reason
  code; tables and per-state geometry at
  [howsteep.org/data/decisions](https://howsteep.org/data/decisions/), the
  full GeoPackage on this repository's [Releases](../../releases) page) —
  ODbL 1.0; see [DECISIONS.md](DECISIONS.md);
- **the OCL spec** — CC BY 4.0; **the JSON Schema** — MIT;
- **the site as a whole** (its selection, arrangement and complete
  corpus) — not licensed; for bulk use beyond the ODbL dataset,
  [get in touch](https://howsteep.org/report?private=1) (a private
  message to the editor; no account needed).

Details and attribution guidance:
[howsteep.org/licensing.html](https://howsteep.org/licensing.html) ·
[LICENSE.md](LICENSE.md).
