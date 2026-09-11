# The decisions dataset

HowSteep publishes not only the climbs it ranks but every road it ever considered
and why each one stands where it does. A climb missing from the rankings is a
decision, not an oversight, and this dataset lets anyone check the decision.

Current release: **[howsteep.org/data/decisions/](https://howsteep.org/data/decisions/)**
(tables, per-state geometry, this registry). The complete GeoPackage of every
line is on the [Releases](../../releases) page. Licence: ODbL 1.0, like the rest
of the survey's data.

## How a road becomes a climb page

1. **Discovered.** Discovery scans the open road network against a coarse
   terrain model and proposes routes worth measuring. Not every proposal is
   measured: each survey run works in bands of coarse steepness, and the rest
   wait for a later run.
2. **Measured.** A batch measures the route on 1 m LiDAR and produces a
   gradient ladder. The ladder is judged against the **adoption doors** — fixed
   thresholds a route must clear to be published: *clean* (a steep 25 m maximum
   with no quality flags), *caveated* (steep but flagged, still plausible),
   *qualified* (a lower bar with the very short windows checked), *endurance*
   (1.5 km or more at 10 percent or more), *sustained* (500 m or more of gain
   at 5 percent or more overall — the alpine-pass class, added 2026-09-10),
   *big-gain* (among the largest rises in its region),
   *listed* (named in a recognised source: a guidebook, a championship, a
   record). A route that clears no door is never published.
3. **Checked against the road authority.** Before adoption a run of checks asks
   whether the measured line is really a public road and really where the road
   is: the state or national road inventory must cover the route, the steepest
   window must sit on the inventoried line, the LiDAR point cloud must show a
   road bed under it, the line must not jog off the road or cross water, and a
   road audit of the OpenStreetMap tags must not object.
4. **Adopted, listed or held.** A route that clears a door and every check is
   **adopted** and ranked. One that clears a door but leaves a doubt a machine
   cannot settle is **listed**: published with its measurements, held out of
   the rankings, marked with the reason, until a person rules. Some rules were
   introduced after pages were already live; when a rule catches an existing
   page it is either trimmed and re-measured, listed, or **retired** with a
   dated ledger entry. A page found to be the same climb as another is merged
   (**duplicate**).

## The files

| File | Rows | Contents |
|---|---|---|
| `decisions.csv` | 8,996 | every page ever published: adopted 7,355, listed 274, retired 1,345, duplicate 22 |
| `never-paged.csv` | 139,524 | routes measured in a survey run that never became a page, one row per run |
| `never-measured.csv` | 16,633 | routes proposed by discovery that no batch measured |
| `codes.csv` | | the reason-code registry below |
| `decisions.gpkg` | | the line of every row that has one, WGS 84, one layer per stage (Releases page) |
| `geojson/` | | the same lines as gzipped GeoJSON, split by layer, section and state |
| `manifest.json` | | counts by stage and code, SHA-256 per file, what could not be located |

Counts are those of the 2026-09-10 release; `manifest.json` in each release is
authoritative.

## Columns

`route_id` — `section/slug@event`; the event is `corpus` for a live page, the
retire ledger for a withdrawn one, `redirect` for a merge, or the survey run for
a route that never had a page. `section` (uk / us / ch), `state` (a US state or a Swiss canton), `run`, `stage`
(adopted, listed, retired, duplicate, measured, discovered), `status_now`
(ranked, annex, listed, withdrawn, never-paged, never-measured), `reason_code`
(one code from the registry), `rule_id` (the survey's internal rule or ruling
identifier), `decided_on`, `decided_by` (*machine*, or *human* when a
road-review verdict or an editor's ruling applied), `evidence` (JSON — only the
survey's own numbers: gradients, lengths, inventory coverage, door names, the
verdict word), `supersedes` (the row this decision replaced), `geometry_ref`,
`slug`, `name`. Human verdicts publish the verdict word only, never the review
text. Nothing from restricted sources appears anywhere in the dataset.

## Reason codes

Codes about **whether the route cleared the bar**

| Code | Meaning |
|---|---|
| `door-cleared` | cleared an adoption door and is ranked; the door is in `evidence` |
| `recognition-listed` | adopted or kept because a recognised source names the climb |
| `authority-lift` | a hold lifted because the road inventory carries the route as public road |
| `no-door` | measured, cleared no door |
| `below-door-at-fold` | re-measured and fell below every door, with a stated reason |
| `not-measurable` | the batch could not measure it: no 1 m ground, a data gap, a failed profile |
| `not-selected` | proposed by discovery, never measured (outside the run's bands) |

Codes about **whether it is a public road**

| Code | Meaning |
|---|---|
| `not-state-road` | the road inventory covers less than half the route |
| `not-municipally-maintained` | no public authority maintains it (absent from New York's Local Highway Inventory; a Pennsylvania segment with no maintained miles) |
| `private-road` | the road audit or inventory marks most of the route private |
| `road-audit-flag` | the OpenStreetMap audit could not vouch for it (unnamed service way, a gate without an access tag, a quarry); listed for a look |
| `code-named-road` | the inventory names the road only by a code; whether it is public is open |
| `human-verdict` | a person ruled on the road; the verdict word is in `evidence` |

Codes about **whether the measured line is on the road**

| Code | Meaning |
|---|---|
| `extent-band-hold` | the inventory covers 50–80% of the route; held for trimming to the covered run |
| `extent-trimmed` | re-measured on the run the inventory covers or names, adopted at the shorter extent |
| `extent-trim-no-door` | without its uninventoried tail the route clears no door |
| `hunt-extent-named-run` | the named road covers less than 80% of the page; held until trimmed |
| `max-off-inventory-tail` | the steepest 25 m sits on a tail beyond the inventoried road |
| `max-off-inventory-not-inventoried` | the steepest 25 m lies off every inventoried road |
| `max-window-offset` | the steepest 25 m sits more than 15 m from any inventory line; held until trimmed |
| `driveway-at-window` | a driveway, service way or track meets the route inside the steepest 25 m while that window sits off the inventory line |
| `line-offset` | the traced line sits off the road bed the LiDAR shows; re-lined or held |
| `prism-absent` | the LiDAR point cloud shows no road bed under the line |
| `displaced-span` | the line jogs off the road inside the steepest window |
| `line-through-buildings` | the mapped line runs through building footprints |
| `over-water` | the line crosses open water |
| `summit-trim` | the machine-built extent ran past the summit and was ended at the top |

Codes about **the measurement itself**

| Code | Meaning |
|---|---|
| `terrarium-second-surface` | a coarse second terrain model disagrees with the LiDAR at the maximum (canopy or structure) |
| `power-crossing` | an overhead line crosses the steepest window and the raw profile spikes there |
| `surface-window-unmatched` | the steepest window could not be matched to a surface record; held for a look |

Codes about **duplicates and gates**

| Code | Meaning |
|---|---|
| `duplicate-of` | the same climb as an existing page; `evidence` says which |
| `dedupe-near-published` | within 100 m of a published route; not adopted until a person rules |
| `duplicate-check-pending` | shares part of its line with another page; listed until ruled |
| `adopter-not-placed` | cleared a door, nothing excluded it, yet no page exists: refused or renamed by the final adoption step (a duplicate, an audit refusal, under 200 m, over the top). The exact gate is not yet coded — the one honest bucket in the dataset |

## What is not in it

The coarse discovery grid itself (cells, not routes), the per-metre elevation
profiles (each climb page links its own OCL document), and anything from
sources the survey may compare against privately but not publish. The rules
are described in the [methodology](https://howsteep.org/methodology).
