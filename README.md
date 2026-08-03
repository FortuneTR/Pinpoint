# Pinpoint

**An interactive atlas of the United States.** Zoom a satellite map, drop onto any state,
and read its story — when it joined the Union, where its name came from, and the historic
sites worth standing in front of. Click a site (the Statue of Liberty, the Alamo, the Lincoln
Memorial) for the facts; every historic site in that state lists down the right-hand panel.

Covers all 50 states **and Washington, D.C.**

---

## What it does

- **Satellite basemap** with smooth zoom — a Google-Earth-style feel, using free imagery tiles.
- **State dossiers** — pick a state and get its statehood date (`MM/DD/YYYY`), admission order,
  and name origin, shown as a clean instrument-style readout.
- **Historic sites on the map** — selecting a state drops pins for its landmarks and fills a
  side list. Click a pin *or* a list row to read the site's story and fly the camera to it.
- **Back to the U.S.** — zoom back out and the state markers return.

The prototype (`index.html`) is fully working: state data is seeded for all 51 jurisdictions,
and historic sites are seeded for California, Florida, Massachusetts, New York, Pennsylvania,
Texas, and D.C. so you can feel the full interaction end-to-end.

---

## The honest challenge: data at scale

The hard part of Pinpoint isn't the map — it's the data. Three layers, three very different sizes:

| Layer | Rough count | Strategy |
|---|---|---|
| **States + D.C.** | 51 | Hand-author once. Small, finite, done in `data/states.js`. |
| **Cities** | ~19,000 incorporated places | **Don't hand-write.** Pull from the U.S. Census / Wikidata; author facts only for major cities, generate the rest. |
| **Historic sites** | ~95,000 (National Register alone) | **Don't hand-write.** Pull from the National Park Service API + Wikidata; curate the marquee ones. |

So the plan is **seed + feed**: curate the small stuff by hand, ingest the big stuff from free
public APIs, and cache it locally so the app stays fast and works offline.

### Data sources (all free)

- **National Park Service API** — parks & historic sites with descriptions and coordinates.
  Free API key. https://www.nps.gov/subjects/developer/
- **Wikidata / Wikipedia REST API** — landmarks, founding dates, name origins, coordinates.
- **U.S. Census (Gazetteer / TIGER)** — authoritative place names, coordinates, and boundaries
  for states and cities. https://www.census.gov/geographies/
- **National Register of Historic Places** — the definitive list of U.S. historic sites.

> ⚠️ **Seed data is best-effort.** The statehood dates and name origins in `data/states.js`
> are a starting point — verify against the Census and state archives before treating any of it
> as authoritative, and cite sources in the UI.

---

## Tech stack

- **[Leaflet](https://leafletjs.com/)** — the map engine (open-source, no API key).
- **Esri World Imagery tiles** — free satellite basemap + a boundaries/labels reference layer.
- **Vanilla HTML / CSS / JS** — no build step; the whole prototype is one file.
- Fonts: Fraunces (display), Inter (body), IBM Plex Mono (data readouts).

**Why no framework yet?** A static site deploys free on GitHub Pages and is dead simple to
reason about. When the data pipeline arrives you can bolt on a small **Flask** backend (or just
pre-generate JSON) without rewriting the front end.

**Optional upgrade path:** swap Leaflet for **Mapbox GL JS** if you want a true 3D globe / tilt
("Google Earth" proper). It needs a free token and vector tiles, but the data model here carries over.

---

## Repo structure

```
Pinpoint/
├── index.html            # Self-contained prototype (data inlined) — just open it
├── data/
│   └── states.js         # Modular version of the dataset (50 states + D.C. + sites)
├── css/                  # (future) extracted styles
├── js/                   # (future) extracted app logic
├── scripts/              # (future) data-ingest scripts: NPS, Wikidata, Census → JSON
└── README.md
```

`index.html` ships self-contained so it previews anywhere. `data/states.js` is the same data as
an external module — switch to it once the dataset grows past what's comfortable to inline (just
add `<script src="data/states.js"></script>` back and delete the inline block).

---

## Roadmap

**Phase 1 — Prototype (done).** Working map, all-states dossiers, seeded sites, side panel.

**Phase 2 — Real state polygons.** Load Census/GeoJSON boundaries so clicking anywhere inside a
state selects it, not just the center pin. Highlight the selected state's shape.

**Phase 3 — Sites at scale.** Write `scripts/ingest_nps.js` to pull NPS + Wikidata into
`data/sites.<state>.json`, cached per state and loaded on demand. Add source links per site.

**Phase 4 — Cities.** Add a city zoom tier: below a zoom threshold, show major-city pins with
founding date and name origin (Census + Wikidata). Facts hand-curated for big cities, generated
for the rest.

**Phase 5 — Polish.** Search box ("jump to a state or site"), share-a-link deep URLs
(`?state=NY&site=statue-of-liberty`), a timeline scrubber that filters sites by era, and a
"random landmark" button.

---

## Run it locally

No build step. Just open the file:

```bash
# clone
git clone https://github.com/FortuneTR/Pinpoint.git
cd Pinpoint

# open directly, or serve it (nicer for the modular data file):
python3 -m http.server 8000
# then visit http://localhost:8000
```

## Deploy free on GitHub Pages

1. Push to `main`.
2. **Settings → Pages → Source: `main` / root.**
3. Your atlas goes live at `https://fortunetr.github.io/Pinpoint/`.

Because it's a static site, that's the whole deployment.

---

## Attribution

- Map imagery © Esri, Maxar, Earthstar Geographics (World Imagery service).
- Map tiles rendered with Leaflet.
- Historic-site and place facts should credit their sources (NPS, Wikidata, Census) as data is added.

## License

Add a license before publishing (MIT is a common, permissive choice for projects like this).
