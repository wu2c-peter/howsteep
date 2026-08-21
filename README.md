# HowSteep

**[howsteep.org](https://howsteep.org)** — reproducible LiDAR gradient
ladders for British road climbs. Every maximum states its measurement
distance, because a hill can honestly be 32% over 2 m and 25% over
100 m at once.

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

## The OCL format

The site's measurements are published in **OpenClimb Ladder (OCL)** — a
data format in which a maximum gradient *cannot be expressed without its
measurement distance*. The schema forbids windowless maxima; that is the
point. Spec: [spec/OCL.md](spec/OCL.md) · normative JSON Schema:
[schema/ocl-climb.schema.json](schema/ocl-climb.schema.json) · live
example: any climb page on [howsteep.org](https://howsteep.org) links its
own OCL document. Comments and implementations welcome — open an issue.

## Data & licences

Measurements are derived from open data: national LiDAR programmes of
all four UK nations (OGL v3), OS Terrain 50 / Open Names / Open Roads
(OGL v3), and OpenStreetMap (ODbL). What the survey publishes, you may
reuse — the terms are settled:

- **any individual climb's page, charts and figures** — CC BY 4.0,
  with credit to "UK Climb Survey — howsteep.org";
- **the complete dataset** (CSV and per-climb OCL documents,
  downloadable from [howsteep.org](https://howsteep.org)) — ODbL 1.0
  (attribution + share-alike, as inherited from OpenStreetMap);
- **the OCL spec** — CC BY 4.0; **the JSON Schema** — MIT;
- **the site as a whole** (its selection, arrangement and complete
  corpus) — not licensed; for bulk use beyond the ODbL dataset,
  [ask](../../issues/new/choose).

Details and attribution guidance:
[howsteep.org/licensing.html](https://howsteep.org/licensing.html) ·
[LICENSE.md](LICENSE.md).
