# Trials Result Protocol

Draft 0.1 of a JSON format for trials results. Each document holds one published result, in as much detail as the source gives: placings, totals or every section mark.

- `trials-result.schema.json` is the schema (JSON Schema 2020-12).
- `examples/` holds nine real results from Australia, Germany, Norway, the UK and the USA.

The examples are real result sheets typed in by hand, to check the format against how clubs actually publish results. Anything that didn't fit is listed in each file's `mapping.misfits`.

## A document

`protocol` (`"trials-result/0.1"`), `document`, `event` and `entries` are required. Objects refer to each other by `id` within the document.

| Key | Holds |
|---|---|
| `document` | The published result: title, status (provisional, official, amended), detail level, publisher, scoring software, source URL. |
| `event` | Name, organiser, governing bodies (highest authority first), country, region, venue, dates, discipline, series. |
| `rule_sets` | How scoring works: what a section value means, whether low or high wins, penalties, time rules, gate values, tie-break order. An event can have several. |
| `routes` | Lines through the sections, usually by colour. `order` 1 is the hardest. |
| `sections` | What is ridden and scored: observed sections, timed tests, speed sections. |
| `parts` | The event's structure as a tree, using `parent`: days, laps, loops, heats, finals, tests. A two-day trial has two `day` parts with `lap` parts under them. |
| `classes` | What riders enter. A class can name its route, lap count and rule set. |
| `entries` | The competitors: `solo`, `crew` (sidecar), `pair`, `team` or `unidentified`, with their people, machine, standing, scores and totals. |
| `classifications` | Every ranked list the document prints. Each row points at an entry, and one entry can sit in several lists. |
| `awards` | Trophies that are not a ranked list. |
| `officials` | Observers, clerk of the course and others, when printed. |
| `mapping` | Notes from whoever transcribed the sheet: source file, how many riders were encoded, the features the sheet shows, misfits (what the protocol couldn't hold cleanly, with a suggested fix) and confidence. |

## Scores

Scores sit on the entry in three forms. Use the finest the source prints.

- `grids`: one row of values per part, in section order. A value is a number, `null` (not ridden) or the text as printed (`"-"`, `"MS"`).
- `rides`: one object per attempt, for anything more than a plain value: a time, a code, gates, a state such as `missed`.
- `totals`: lap, day and event totals. `part: null` is the event total. A total splits into `observation`, `time_marks`, `penalties` and `adjustments`, with `total` as printed. `counts` holds printed mark counts, such as `{"0": 21, "1": 4, "5": 2}`.

One rider from the Tasmanian Classic Trial, trimmed:

```json
{
  "id": "e1",
  "kind": "solo",
  "people": [{"role": "rider", "person": {"name": {"given": "Chris", "family": "Bayles", "as_printed": "Chris BAYLES"}}}],
  "machine": {"make": "TRS", "as_printed": "TRS 300"},
  "classes": ["t2"],
  "grids": [
    {"part": "l1", "values": [0, 1, 0, 0, 0, 1]},
    {"part": "l2", "values": [0, 0, 1, 0, 0, 0]},
    {"part": "l3", "values": [0, 0, 0, 0, 0, 1]},
    {"part": "l4", "values": [0, 2, 2, 2, 0, 5]}
  ],
  "totals": [
    {"part": "l1", "total": 2},
    {"part": "l2", "total": 1},
    {"part": "l3", "total": 1},
    {"part": "l4", "total": 11},
    {"part": null, "observation": 15, "total": 15, "unit": "marks",
     "counts": {"0": 16, "1": 4, "2": 3, "3": 0, "5": 1}, "status": "classified", "rank": 1}
  ]
}
```

## Encoding rules

- **Standing and status are separate.** `standing` is the basis the rider rode on: competition, non-competitive, no award or recreation. `status` is how the ride ended: classified, DNF, DNS, retired, over time, missed section and so on. A non-competitive rider can be classified, and a competitive rider can DNF.
- **The published rank is the truth.** Store it in `rank`, with `rank_as_printed` when it isn't a plain number (`"1="`, `"RET"`) and `tied: true` for a shared place. Never recompute it.
- **Say why equal scores were split.** When riders on the same score get different places, each gets a `tie_break`: `rule` names the tie-break, `label` is what separated that rider ("32 cleans" for one, "31 cleans" for the other) and `detail` explains it. `inferred: true` means the sheet doesn't print the reason and it was worked out from the data: `{"rule": "most-cleans", "label": "52 cleans", "inferred": true}`.
- **Keep the printed text.** When a value is normalised, the original goes in the nearest `as_printed`. Names always have `as_printed`. `given` and `family` are filled only when the split is certain.
- **Leave out what isn't printed.** A missing field means the source didn't give it.
- **Use `x_` keys for gaps.** Every object accepts keys starting with `x_` for anything the protocol has no place for yet.

## Rider identity

- `entry.number` is the riding number worn on the day. It is not an identity.
- `person.prin` is the Public Rider Identification Number: one id for one rider across every event. Its format is not fixed yet.
- `person.identifiers` holds federation licence ids. A document may carry them only when `document.distribution` is `internal`.
- `rider_ride_uuid` identifies one person's ride at one event.

The schema checks structure only. It does not check that a referenced `id` exists, or the `identifiers` rule.

## Examples

Each example holds every rider on its source sheet. `document.source_url` links the original where one is public.

| File | Country | Shows | Riders |
|---|---|---|---|
| `2026-07-11_natc_mototrials-colorado-day-1-pro_results.json` | USA | AMA/NATC national, Pro class, day 1: three laps of section marks. | 5 |
| `2026-10-17_richmond-motor-club_scott-trial_results-sections.json` | UK | Scott Trial: one 76-section course scored on observation plus time, with over-time, missed-section, DNF and DNS riders. | 195 |
| `2026-04-25_dmsb_deutsche-trial-meisterschaft-r1-ebstorf_results.json` | Germany | German championship round 1: 3 laps of 10 sections, a line colour per class, a late-finish penalty, championship points. Only riders scoring for the championship are listed. | 65 |
| `2025-06-14_nmf_nm-x-trial-qualifying_results.json` | Norway | X-Trial qualifying: men and women ride different sections, penalty points, shared places. | 24 |
| `2025-11-16_tasmanian-motorcycle-trials-club_classic-trial_results.json` | Australia | Club trial: 4 laps of 6 sections, DNF and non-competitive riders. | 24 |
| `2022-04-17_ajs-mototrials-wa_gasgas-easter-fun-trial_trial-1a-results.json` | Australia | Timed tests: each run's time, dabs, fives and tape breaks. The sheet covers 2 of the event's 8 runs. | 26 |
| `2026-09-19_motorcycling-victoria_victorian-trials-championships_results.json` | Australia | Two-day state championship, lap counts that differ by class, non-competitive riders. | 49 |
| `2026-10-10_motorcycling-australia_australian-trial-championships_results.json` | Australia | Two-day national championship: 14 classes, riding numbers, sidecar crews. | 91 |
| `2016-04-08_motorcycling-australia_australian-x-trial-championships_results.json` | Australia | Indoor X-Trial: qualifying heats, a final ranked on points, a cancelled section. | 8 |

Riders in junior classes appear with their surname shortened to an initial. Scores a publisher hid on the page are not included.
