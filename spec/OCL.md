# OpenClimb Ladder (OCL)

**A data format for road-climb gradient profiles in which a maximum
gradient cannot be expressed without its measurement distance.**

Version: **0.1.0-draft** · Status: draft for comment ·
Reference implementation: [howsteep.org](https://howsteep.org) — every climb page serves its measurements as an OCL JSON document (pipeline code currently private)

---

## Why

Published climb data routinely reports a single "maximum gradient" with no
definition of the distance over which it was measured. Such numbers are
unfalsifiable: 32% and 25% can both be "correct" for the same hill at
different measurement scales. OCL makes the scale part of the value. A
conforming document *cannot* contain a windowless maximum — the schema
forbids it.

## Core concepts

**Ladder.** The gradient profile of a climb summarized as maxima over a
set of explicit plan-distance windows, in metres:

| Window | Role |
|---|---|
| 2, 3 m | *diagnostic* — raster-noise scale; published with caveat, never ranked |
| 5, 10, 20, 25, 50, 100 m | *standard* — headline and ranking eligible |
| 200, 500, 1000 m | *extended* — optional, for long climbs |

**Plan distance.** All windows are horizontal distances along the
projected centreline, not 3-D surface distances. On a curve the window
follows the arc of the centreline — a window is w metres of path, not
a w-metre straight chord.

**Measurement line.** The line MUST be the carriageway centreline.
Producers SHOULD declare the line's source and SHOULD verify its
fidelity against an independent road-network dataset: a line that cuts
the inside of a bend under-runs the true arc and samples ground the
road does not occupy, inflating every window that lands there.

**Window gradient.** `gradient(d, w) = 100 × (elev(d+w) − elev(d)) / w`,
a forward window on a uniform sampling grid. Overall gradient uses
endpoint elevation difference, never accumulated ascent.

**Claims vs measurements.** Externally reported values (road signs,
databases, GPS platforms, record adjudications) are *claims*, carried in a
separate structure with evidence type, source and provenance credibility.
They are never merged into measured fields.

## Name

`name` is the climb's display name — the string a page's heading shows. It
is NOT a stable identifier: `slug` is, and the slug never changes when the
name does. A producer MAY derive `name` from any of:

1. the road or feature's own name (a named pass, a named summit approach,
   or any climb identified as an already-named feature by route-line
   distance to an authoritative coordinate);
2. for a climb running along more than one named street, the OSM street
   holding the steepest 25 m window, when that street is a real part of
   the climb (a majority of the window's own length, and at least 20% of
   the climb's own length on that street) — never for a street that merely
   crosses or briefly coincides with the line;
3. for an otherwise-unnamed lane, a name built from the nearest OpenStreetMap
   place anchor: "Lane near `<place>`", or "`<street>` (`<place>`)" when (2)
   also names a street for it.

A name derived under (1) from a recognition or identification-certain
source is never displaced by (2) or (3) once set.

**Open item (draft, not yet normative):** this document does not define a
field that records which of the above produced a given `name`, or what OSM
element(s) it came from. Today's reference implementation keeps that
history only in its own internal override ledger, not in the published OCL
document — a consumer reading `name` alone cannot tell a recognised pass
name from a steepest-street or nearest-place name, or audit it against the
source. A future minor revision should consider a `name_source` (or similar)
provenance field, analogous to `reported_claims`' `evidence_type`, before
this draft is finalised.

## Document structure (informative summary)

Normative schema: [`schema/ocl-climb.schema.json`](../schema/ocl-climb.schema.json).

```jsonc
{
  "ocl_version": "0.1.0",
  "name": "Cefn Llan",
  "slug": "cefn-llan",
  "length_m": 218.5,
  "endpoint_gain_m": 42.4,
  "overall_gradient_pct": 19.41,
  "ladder": [
    {"window_m": 2,  "max_gradient_pct": 32.05, "diagnostic": true,
     "start_distance_m": 61.0, "end_distance_m": 63.0},
    {"window_m": 25, "max_gradient_pct": 27.38, "diagnostic": false,
     "start_distance_m": 41.0, "end_distance_m": 66.0}
    // ... one entry per window, ascending
  ],
  "confidence": "A",               // measurement confidence A/B/C/D/U —
                                   // NOT a gradient category; see below
  "provenance": {
    "analysis_version": "…",
    "road_source": "OpenStreetMap",
    "osm_ids": [218612917, 103333437],
    "elevation_provider": "Welsh Government",
    "elevation_resolution_m": 1,
    "elevation_crs": "EPSG:27700",
    "sample_interval_m": 1,
    "interpolation": "bilinear",
    "smoothing": "none"
  },
  "reported_claims": [             // optional; claims, not measurements
    {"source_name": "Road sign (current)", "value_pct": 32,
     "evidence_type": "road_sign", "claimed_window_m": null,
     "observed_via": "osm_tag_history",  // optional: how the observation
                                         // reached the producer — a sign
                                         // documented via OSM history is
                                         // strong evidence, but not a
                                         // photographed sign
     "credibility": "…"}
  ]
}
```

## Normative rules

1. **R1 — window-mandatory.** Every maximum-gradient value MUST appear
   inside a ladder entry with a `window_m`. There is no top-level
   `max_gradient` field and conforming consumers MUST reject documents
   that add one.
2. **R2 — diagnostic separation.** Entries with `diagnostic: true` MUST
   NOT be used as headline values or ranking keys by consumers.
3. **R3 — provenance-mandatory.** `provenance` MUST identify road-geometry
   source, elevation source and resolution, CRS, sample interval,
   interpolation, and analysis software version.
4. **R4 — claims separation.** Third-party values MUST appear only under
   `reported_claims`, each with an `evidence_type`. Producers MUST NOT
   copy claims into measured fields.
5. **R5 — endpoint overall.** `overall_gradient_pct` MUST be computed
   from endpoint elevation difference over plan length.
6. **R6 — display precision.** Displayed precision SHOULD be
   consistent with window uncertainty: whole percent for windows of
   10 m and below (raster noise there is >=1 pp), one decimal place
   for longer windows and overall values. Stored values MAY carry full
   precision.
7. **R7 — terrain labelling.** DTM-derived values MUST NOT be described
   as surveyed road-surface gradients.

## Measurement confidence

An overall statement of how much trust to place in the measurement,
carried in `confidence`. Deliberately NOT called "grade" or
"category" — in cycling both words mean the gradient itself; this
letter describes the *evidence for the numbers*, never the hill.
Two axes: source resolution and how much quality intervention the
standard (ranking-eligible) windows needed.

- `A` — ≤1 m source, standard windows clean: no canopy/structure flags,
  no re-sited maxima, nothing unresolved.
- `B` — source in (1 m, 2 m], or a ≤1 m source with interventions on
  standard windows (canopy/structure flags, re-sited maxima).
- `C` — source in (2 m, 5 m], or standard windows left unresolved by
  suppression (no clean placement of the window existed).
- `D` — source coarser than 5 m; short-window maxima suppressed.
- `U` — open human-review flags: the numbers await scrutiny.

Extent flags do not affect confidence — they question the climb's
definition, not the measurement. Rankings SHOULD default to A/B.

### Provenance run identity (added 2026-08-22, additive)

`provenance.run_id` (string, optional): identifier of the batch run
that produced the document, stamped by the producing pipeline.
Publishers SHOULD record it so a document's station-level evidence
remains addressable; consumers MUST NOT assign it meaning beyond
opaque run identity. Additive within 0.1.x (`provenance` permits
additional properties).

### Provenance run manifest (added 2026-09-19, additive)

`provenance.run_manifest` (object of strings, optional): the run that
produced the document, in enough detail to find the code and the inputs
again. Every value is a string; the block is flat and small. Recognised
keys, none of them required:

| key | what it names |
|---|---|
| `manifest_version` | shape of the manifest (`"1"`) |
| `code_git_sha`, `code_git_dirty` | the producing checkout's commit, and whether its source tree differed from that commit. `code_git_sha: "unknown"` with a `code_git` sentence means the producer was not running from a version-control checkout — it MUST NOT be read as a clean checkout |
| `analysis_version` | the producing software's release version (the same value as `provenance.analysis_version`) |
| `analysis_config_digest` | `sha256:<hex>` over the canonical analysis configuration actually in force — window ladder, QA and mask thresholds, sample interval, interpolation |
| `python`, `numpy`, `rasterio`, `gdal`, `proj`, `pyproj`, `laspy`, `shapely`, `pyosmium`, `libosmium` | versions of the numerical libraries the run linked. An absent key means the library was not part of the run |
| `region_key`, `crs`, `elevation_provider_key`, `tile_index`, `tile_index_snapshot` | the registry identity, working CRS, elevation provider, and a digest of the tile index the run could see |
| `road_pbf`, `road_snapshot`, `authority`, `authority_files` | the road-geometry extract and the road-authority dataset(s) the run read |
| `run_id` | the batch run identity (also carried as `provenance.run_id`) |

Producers SHOULD record every key they can state truthfully and MUST
omit the rest rather than publish a blank or a guess. Consumers MUST NOT
infer anything from an absent manifest beyond its absence: documents
published before 2026-09-19 have none and remain valid.

## Versioning

Semantic versioning of the format itself, carried in `ocl_version`.
Breaking schema changes bump the major version. This draft is 0.1.0:
everything is open for comment.

## Licence and attribution

The OCL specification is released under CC BY 4.0; the normative JSON
Schema is MIT-licensed. Datasets published in OCL carry their own
licences; documents derived from OpenStreetMap geometry inherit ODbL
obligations, and producers MUST retain the attribution fields in
`provenance`. Producers SHOULD declare the document's licence in
`provenance.license` (an SPDX identifier where one exists, e.g.
`ODbL-1.0`) and MAY point `provenance.license_url` at a page with the
full terms, so each document carries its own terms when it travels.
