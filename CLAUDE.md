# CLAUDE.md

## What this is

A **historical-geography research project**, not a software product. It maps a proposed
route for the Moses / Musa narrative (Levant → Wadi Arabah → NW Arabia) as GeoJSON layers,
presented as a scroll-driven atlas over a real terrain basemap.

No build system, no package manager, no tests, no git repo. Edit the files and reload.

**All authored content is Arabic (RTL).** Property values, narrative text and UI strings
stay Arabic; property *keys* stay English snake_case. **Numerals are Western digits
everywhere** — in the data, in the narrative text, and on screen. There is no numeral
conversion; do not reintroduce one, and do not hand-type ٤٠٢ into a file.

### Naming the prophet

**Always write نبي الله موسى, never موسى alone — everywhere, including place names.**
This is the researcher's explicit decision (2026-08-29) and applies without exception:
the atlas title, evidence and notes, and the toponyms — وادي نبي الله موسى،
عين نبي الله موسى، عيون نبي الله موسى، آبار نبي الله موسى.

Note that these differ from the registered map names (Wadi Musa is the town at Petra's
entrance), so the gazetteer intentionally does not match official cartographic naming.

When adding text, apply the honorific by *phrase* and check for
`نبي الله نبي الله` afterwards — a blind search-and-replace on موسى double-applies it
to strings that already carry the honorific.

## Running it

The app reads neighbouring files, so `fetch` is blocked under `file://` — a double-click
open shows an Arabic message explaining exactly this. Serve the folder:

```bash
cd Profet-mosa
python3 -m http.server 8000
open http://localhost:8000/
```

Tiles need internet. Everything else (Leaflet, data) is local.

## Layout

The project is exactly 13 files. Every one of them is loaded at runtime except this note.

| Path | Role |
|---|---|
| `index.html` | The app: Leaflet + terrain tiles, six scroll-driven events, site sheet, gazetteer index, all styling |
| `data/narrative.json` | The prose: title, intro, the six events (title, dek, notes, colour), outro |
| `last/99_Gazetteer.geojson` | The 44 sites — source of truth for coordinates, tiers, evidence |
| `last/01`–`06_*.geojson` | Route geometry, one file per event |
| `last/00_Ancient_Roads.geojson` | Ancient roads drawn as faint context under every event |
| `vendor/leaflet.{js,css}` | Leaflet 1.9.4, vendored so the app needs no CDN |
| `CLAUDE.md` | This file |

Earlier drafts (the static `atlas.html` plate viewer, `moses/`, `maps/`, `images/`, the
root-level draft GeoJSON/KML and `00B_Egyptian_Presence.geojson`) were deleted on
2026-08-29. They held no site the gazetteer does not already have — only superseded
coordinates for the same places. They were archived to a tarball outside the repo before
deletion; that archive is local to the author's machine and is not part of this project.

## The evidence-tier vocabulary (the core discipline)

Every site carries a `tier` stating *how strongly it is attested*. Keeping claim strength
visually separate from geography is the point of the whole project.

- `attested` — archaeologically documented · filled disc
- `inferred` — argued from terrain/route logic · hollow ring
- `toponym` — the modern name preserves the memory · diamond
- `hypothesis` — the researcher's own proposal · dashed ring
- `road` — ancient road LineStrings in `00_Ancient_Roads.geojson`

Rules that hold across the corpus:
- **Tier controls marker shape. Event controls colour.** Never encode a claim in the colour.
- Anything `hypothesis` must carry a `caveat` naming what is not proven.
- Every site needs `evidence`. Never leave one bare.

## Data contract

`99_Gazetteer.geojson` is the **single source of truth** for coordinates, tiers and evidence.
`index.html` builds its whole site table from it at load and holds no site data of its own.

Point properties: `id`, `name`, `short_name`, `modern_name`, `ancient_name`, `subtitle`,
`tier`, `evidence`, optional `caveat` / `role`, plus simplestyle `marker-*` keys.

- `id` — stable ASCII slug (`beisan`, `strait`, `bada`). **Never rename one.**
- `short_name` — the map label, shorter than `name` because it is drawn at ~11.5px.
- `subtitle` — the second line in the site sheet.

Event layers (`01`–`06`) supply the route geometry. A point is drawn as a **numbered
station** if its coordinate is a vertex of one of that layer's LineStrings, otherwise as a
**reference site** — derived from the data, not hand-flagged. Event layers join to the
gazetteer by exact coordinate, so a coordinate typo silently drops a site from the map.

The map frame for each event is computed from that event's own geometry, not from the
`bounds` field in `narrative.json` (kept only as a fallback).

## Invariants — verify after any edit

1. **WGS84 `[lon, lat]`**, identical for the same site in every file. 44 sites, currently
   zero conflicts. Keep it that way.
2. **Crossing direction is west → east**: Palestine → Jordan at the Lisan strait
   (~35.42, 31.14), *not* the Mujib mouth (~400 m deep there). `mujib` is in the gazetteer
   as the documented, rejected alternative.
3. **Joshua's northern ascent follows the King's Highway** over the eastern plateau
   (بصيرة → الكرك → ذيبان → مادبا), not Wadi Arabah.
4. Petra and Jabal Harun are **reference sites**, not stops — ~1000 m above the Arabah floor.

```bash
# every layer parses, every gazetteer id is unique, no stray Arabic-Indic digits
python3 -c "
import json,glob,re
ids=[f['properties'].get('id') for f in json.load(open('last/99_Gazetteer.geojson'))['features']]
print('gazetteer:',len(ids),'unique ids:',len(set(ids)))
for f in sorted(glob.glob('last/*.geojson')):
    print(f, len(json.load(open(f))['features']))
for f in glob.glob('last/*.geojson')+['data/narrative.json','index.html']:
    if re.search('[\u0660-\u0669]', open(f,encoding='utf-8').read()): print('ARABIC-INDIC DIGITS in',f)"
```

## Notes for working on `index.html`

- Fixed overlays (legend, site sheet, marker labels) sit *opposite* the narrative column —
  a physical intent, so they use `left`/`right`, not `inset-inline-*`, which flip under RTL.
- **Typography** is a three-family system, each with one job:
  `--display` Amiri (headings), `--body` Noto Naskh Arabic (all prose — naskh is the most
  legible style for running Arabic), `--ui` IBM Plex Sans Arabic (chrome, chips, map labels,
  numerals). Do not use the UI face for paragraphs or the body face for controls.
  Arabic prose needs air: body copy runs 14.5–15.5px at line-height 1.95–2.05.
- `zoomSnap:.5` is a deliberate balance. At fully fractional zoom raster tiles get CSS-scaled
  and the basemap labels soften; at integer-only snapping `fitBounds` rounds down and wastes
  up to a third of the frame. Half-steps keep both acceptable.
- `--now` carries the active event's colour to the progress bar and focus rings; it is set
  by the scroll observer, so anything that should follow the event colour reads that var.
- If you change the map-label font size, update the width factor in `declutter()` — it
  estimates each label's box from character count.
- `declutter()` tries each label on the left of its dot, then the right (`.lb.alt`), before
  hiding it. The honorific makes labels long, and two-sided placement is what keeps them
  on the map — do not reduce it to one side.

### Two kinds of distance — keep them distinct

A card can show two different numbers for the same journey, and they are **not** in conflict:

- the `dek` in `narrative.json` carries the researcher's **overland estimate** (event 2:
  نحو 620 كم), which accounts for real walking over terrain;
- the chip carries the **measured length of the drawn polyline** (event 2: 510 كم), summed
  from station to station.

The chip is therefore labelled `كم على الخط المرسوم` — "along the drawn line" — so the
smaller figure reads as a different measurement rather than a contradiction. A straight
polyline always underestimates a real route; a ~20% gap is expected, not a bug.

`routeKm()` skips tentative lines (`isDashed`), and the chip is omitted when the result is
zero. That is why event 5 shows no distance: the wandering is a zone, and its only line is
the possible Khaybar extension — measuring it would contradict the card's own note that
التيه يُرسم نطاقاً لا خطاً.
- Labels are decluttered greedily every zoom/move — stations first, then reference sites.
  Anything that cannot find room is hidden rather than overlapped.
- The route draw-on is decoration: it clears its own dash after 1.7s and is skipped under
  `prefers-reduced-motion`, so the line is never invisible if the transition does not run.
- Solid routes get a dark casing for contrast; **dashed routes must not** — the casing shows
  through the gaps and turns a tentative line into a solid grey one.
- Event colours live in `narrative.json`. Event 5 (the wandering zone) is teal `#0E8A8F`:
  the earlier gold washed out against the beige relief. Gold `#D4A62A` is reserved for
  correction notes and caveats, which is a separate, semantic use.

## Working style

- This is contested research. Keep the researcher's framing: never silently upgrade a
  `hypothesis` to a fact, and never delete a `caveat` to make a claim read cleaner.
- Narrative prose belongs in `data/narrative.json`; site evidence belongs in the gazetteer.
  Neither belongs in the code.
