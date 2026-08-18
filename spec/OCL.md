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
projected centreline, not 3-D surface distances.

**Window gradient.** `gradient(d, w) = 100 × (elev(d+w) − elev(d)) / w`,
a forward window on a uniform sampling grid. Overall gradient uses
endpoint elevation difference, never accumulated ascent.

**Claims vs measurements.** Externally reported values (road signs,
databases, GPS platforms, record adjudications) are *claims*, carried in a
separate structure with evidence type, source and provenance credibility.
They are never merged into measured fields.

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

## Versioning

Semantic versioning of the format itself, carried in `ocl_version`.
Breaking schema changes bump the major version. This draft is 0.1.0:
everything is open for comment.

## Licence and attribution

The OCL specification is released under CC BY 4.0. Datasets published in
OCL carry their own licences; documents derived from OpenStreetMap
geometry inherit ODbL obligations, and producers MUST retain the
attribution fields in `provenance`.
