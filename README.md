# مسار نبي الله موسى · Masar Nabi Allah Musa

**A research atlas of a proposed route for the Moses / Musa narrative — Levant → Wadi Arabah → north-west Arabia.**

Six events, one route. Every station sits at its real WGS84 coordinate over actual terrain,
and every site carries its **evidence tier** openly: what is archaeologically attested, what
is inferred from terrain, what survives only as a place name, and what is the researcher's
own hypothesis.

The interface is Arabic (RTL). Research and cartography by عبد الواحد (Abdulwahed Mansour).

---

## This is a hypothesis, not settled history

This atlas argues a specific, contested reading of the geography — most notably that the sea
crossing took place west → east at the shallow Lisan strait of the Dead Sea. That claim, and
others like it, are marked `hypothesis` in the data and carry an explicit caveat naming what
is *not* proven.

The whole design exists to keep those categories apart. **Marker shape encodes claim
strength; colour encodes which event.** A reader should never have to guess whether a point
is an excavated site or a proposal.

## Evidence tiers

| Tier | Arabic | Marker | Meaning | Sites |
|---|---|---|---|---|
| `attested` | مثبت أثرياً | filled disc | Archaeologically documented | 27 |
| `inferred` | مرجّح تضاريسياً | hollow ring | Argued from terrain and route logic | 9 |
| `toponym` | اسم يحفظ الذكرى | diamond | The modern name preserves the memory | 3 |
| `hypothesis` | فرضية الباحث | dashed ring | The researcher's own proposal | 5 |

Every site carries an `evidence` note. Every `hypothesis` additionally carries a `caveat`.

## The six events

| # | Event | Extent |
|---|---|---|
| 1 | الولادة والإلقاء — Birth and the river | Beisan and the Jordan valley |
| 2 | الهروب إلى مَديَن — Flight to Midian | South via the highlands, then Wadi Arabah |
| 3 | العودة والتبليغ — Return and the message | Al-Bada' → Jerusalem → Beisan |
| 4 | الخروج والعبور — Exodus and the sea crossing | Beisan south, crossing at the Lisan strait |
| 5 | أرض التيه — The wandering | A zone, not a line, reaching Khaybar |
| 6 | قيادة يوشع — Joshua's leadership | North on the King's Highway, then Jericho and Jerusalem |

Two corrections are recorded in the atlas itself: the crossing is fixed at the Lisan strait,
**not** the Mujib mouth (~400 m deep there); and the northern ascent follows the King's
Highway over the eastern plateau, **not** Wadi Arabah.

## Running it

The page reads neighbouring data files, so `fetch` is blocked under `file://` — opening the
HTML by double-click shows a message saying exactly that. Serve the folder instead:

```bash
git clone <this-repo> && cd Profet-mosa
python3 -m http.server 8000
open http://localhost:8000/
```

Map tiles need an internet connection. Everything else — Leaflet, all data — is local.

## Structure

```
index.html                 the app: map, scroll narrative, site sheet, gazetteer
data/narrative.json        all prose: title, intro, the six events, outro
last/99_Gazetteer.geojson  44 sites — source of truth for coordinates, tiers, evidence
last/01–06_*.geojson       route geometry, one file per event
last/00_Ancient_Roads.geojson   ancient roads, drawn as context under every event
vendor/leaflet.{js,css}    Leaflet 1.9.4, vendored — no CDN dependency
CLAUDE.md                  working notes: data contract, invariants, design decisions
```

No build step, no package manager, no framework. Edit a file and reload.

## How the data fits together

`99_Gazetteer.geojson` is the **single source of truth**. The app builds its entire site
table from it at load and holds no site data of its own — edit the gazetteer and the map
follows.

Event layers supply route geometry. A point is drawn as a **numbered station** when its
coordinate is a vertex of one of that layer's LineStrings, and as a **reference site**
otherwise — derived from the data, never hand-flagged. Layers join to the gazetteer by exact
coordinate, so a coordinate typo silently drops a site.

Each event's map framing is computed from its own geometry, so the view can never drift out
of step with the data.

See [CLAUDE.md](CLAUDE.md) for the full contract, the invariants to check after editing, and
the reasoning behind the design decisions.

## Attribution

- Basemaps — [Esri](https://www.esri.com/) World Shaded Relief, World Hillshade and World
  Imagery; [OpenTopoMap](https://opentopomap.org/) (CC-BY-SA) over
  [OpenStreetMap](https://www.openstreetmap.org/copyright) data. Subject to their terms.
- [Leaflet](https://leafletjs.com/) 1.9.4 — BSD-2-Clause.
- Typefaces — Amiri, Noto Naskh Arabic, IBM Plex Sans Arabic (SIL Open Font License).

## Licence

Not yet chosen. Until one is added, the research content, text and compiled data here are
© Abdulwahed Mansour, all rights reserved.
