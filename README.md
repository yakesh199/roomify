# Roomify — Project Package

Hey — this is everything for the room-customizer / furniture-marketplace project, all gathered into
one place so it's actually easy to find your way around. If you're new here: welcome, and sorry in
advance if a couple of these notebooks still have my messy debugging cells in them.

## Folder structure

```
Roomify_Package/
├── frontend/RoomCustomizerWeb/index.html   Working Three.js prototype (also live on GitHub Pages)
├── data/                                   The catalog + raw source data
├── notebooks/                              The Jupyter notebooks that built the catalog
├── backend/                                Supabase/FastAPI backend scaffolding
└── docs/                                   Design doc + a standalone marketplace UI mockup
```

## frontend/RoomCustomizerWeb/

Just open `index.html` in a browser — double-click it, no server, nothing to install — and you can try
the whole thing: sign up (it's a demo login, so literally any input works) → a fake room scan →
straight into the **My Rooms** dashboard. That dashboard doesn't start empty, on purpose: it comes with
a premade **"Modern Living Room"** already furnished with real catalog pieces (leather sofa, wood
coffee table, recliner, media console, bookcase, ottoman, gold pouf, floor lamp, a dusty-blue accent
wall) — a little "wow" room seeded once on first load, because staring at a blank dashboard on a demo
is nobody's idea of a good first impression. Hit **+ New Room** to name and create one of your own, or
click any room card to open it in the 3D editor. Once you're in the editor, **+ Add** opens the
furniture drawer, which defaults to the **Marketplace** tab — 229 real furniture items in there, with
real photos, real dimensions, and real 3D models pulled live from the Amazon Berkeley Objects dataset.
Click one and it drops into the room as an actual 3D model. There's also a **Basic Shapes** tab with
the original 10 procedural placeholder items, for when you just want something quick and blocky. Move,
Rotate, Replace, Sell, and Delete all work on both kinds of furniture. Rooms autosave to the browser's
localStorage after every single change, not just when you leave the editor — I got tired of losing work
to accidental refreshes, so now it just quietly saves as you go.

The top navbar (**My Rooms** / **Marketplace** / your avatar → **Profile**) follows you everywhere
except inside the 3D editor itself, where it'd just get in the way. **Marketplace** is the real deal: the
full 229-item catalog shown as "New" listings with real photos, plus anything anyone's personally put up
for sale via **Sell** or the sidebar's **List a piece**. Every real-item card has a **+ Add to a room**
button that drops it straight into whichever room you're working on, so the Add Furniture drawer and the
Marketplace actually talk to each other instead of being two disconnected features that happen to share a
catalog. **Profile** shows your name/email (whatever you typed at signup), some stats across all your
rooms, and everything you've personally listed for sale.

You're not locked into one room either — you can keep as many named rooms as you want, and rename or
delete any of them from the hover menu on its card. If you'd saved a single room with an older version
of this app, don't worry, it gets migrated automatically into a room called "My Room" the first time you
open the new version. Nothing gets lost.

Visually I went with a deep indigo accent on a slate/neutral palette, serif headings for a bit of warmth,
and [GSAP](https://gsap.com) handling the little touches that make it feel alive — page-enter transitions,
staggered card-grid reveals, modal pop-ins, button/card hover-press feedback, and the profile stat
count-up. GSAP loads from a CDN, and if that fails for any reason (offline demo, flaky wifi, whatever),
everything just falls back to the underlying CSS transitions instead of breaking. It should never feel
broken, just a little less polished.

I split what used to be one enormous 1260-line `index.html` file into pieces, mostly so future-me (or
whoever reads this next) doesn't have to scroll through a thousand lines to find one function:
- `index.html` — markup only, nothing else.
- `css/styles.css` — every bit of styling; the design tokens live in `:root` at the very top if you want
  to retheme things.
- `js/market-catalog.js` — the 229-item real furniture data. Pure data, no logic hiding in here.
- `js/catalog.js` — the 10 procedural "Basic Shapes" plus the shared catalog/market helper functions.
- `js/scene.js` — the three.js renderer/camera/lights/room setup, the app's `state` object, and
  screen/page switching.
- `js/animations.js` — reusable GSAP helpers (page transitions, grid stagger, modal open, hover/press,
  toast pop, stat count-up); every single helper quietly no-ops if GSAP never loaded.
- `js/rooms.js` — the multi-room data model: create/rename/delete/open a room, the legacy single-room
  migration, and the friendly "edited Xd ago" timestamps.
- `js/furniture.js` — spawning, selecting, moving, rotating, replacing, deleting, and selling furniture.
- `js/input.js` — pointer/raycast handling: click-to-select, drag-to-move, wall picking.
- `js/persistence.js` — saving/loading the active room to and from `state.rooms` (autosaves on every
  mutation, see above).
- `js/ui.js` — login/scan navigation, the navbar, the My Rooms dashboard, the Add Furniture drawer, and
  the Marketplace screen. This one's the biggest file and I know it.
- `js/profile.js` — the Profile page.
- `js/main.js` — the render loop plus app bootstrap; loaded dead last on purpose.

These are all plain `<script src>` tags sharing the global scope — no bundler, no ES modules, nothing
fancy — loaded in dependency order, so the whole thing still works by just double-clicking `index.html`.
No build step, no local server, no `fetch()` of local files (which browsers block over `file://` anyway,
so that wasn't really an option).

One thing worth knowing: it needs an internet connection, since it loads three.js from a CDN and pulls
furniture models from Amazon's S3 bucket the moment you click something.

## data/

- `catalog.csv` / `catalog.json` — 360 cleaned furniture products (id, name, category, brand, color,
  style, dimensions). `catalog.json` is the enriched version, with volume, footprint, room-type tags,
  and a storage score baked in.
- `size_bands.csv` — the S/M/L size-band cutoffs, computed per category.
- `market_catalog.json` — the 229-item subset that actually has a real 3D model, with image + `.glb`
  reference data, shaped exactly the way the web prototype expects it.
- `catalog_with_3d_models.csv` — the same 229 items as plain CSV, in case you just want to poke around
  in a spreadsheet.
- `listings_0_raw_ABO.json` — the raw Amazon Berkeley Objects source data (9,232 listings across every
  product type, not just furniture) that `notebooks/data.ipynb` cleaned everything else from. It's the
  biggest file here by far (~54 MB) and you honestly probably won't ever need to open it — it's kept
  around purely for provenance, so the whole pipeline is reproducible from scratch if someone needs that.

**License note, because this matters:** all of this data (including the 3D models) comes from the Amazon
Berkeley Objects dataset, licensed **CC BY 4.0** — free to use commercially, you just have to credit
Amazon.com. The design doc in `docs/` used to say CC BY-NC, which was wrong; that's been corrected.

## notebooks/

- `data.ipynb` — takes the raw ABO data and turns it into the cleaned `catalog.json`/`.csv`, plus all
  the enrichment work (volume, footprint, room types, storage score, size bands).
- `fitCheck.ipynb` — answers the practical question: does this set of furniture actually fit in a room,
  given ceiling height, footprint in both rotations, and total floor area? Also includes a greedy "what
  should I drop" suggester for when the answer is no.
- `recSystem.ipynb` — the two-stage recommender: first a geometry/room-type filter to weed out anything
  that can't physically work, then need + taste scoring on what's left.

## backend/

Supabase + FastAPI scaffolding, described in more depth in `docs/backend_design_report.pdf`.
- `config.py` / `test_conn.py` — the Supabase client plus a quick connection test (you'll need a `.env`
  with `SUPABASE_URL` / `SUPABASE_KEY` — not included here, for obvious reasons).
- `load_catalog.py` — pushes `data/catalog.json` into a Supabase `products` table.
- `requirements.txt` — fastapi, uvicorn, supabase, python-dotenv, pandas.
- `main.py` — honestly, still empty. The actual API endpoints (`/products`, `/recommend`, `/fit-check`,
  etc., per the design doc) just haven't been built yet. It's on the list.

## docs/

- `backend_design_report.pdf` — the full architecture and API spec, if you want the deeper thinking
  behind the backend.
- `roomscan_demo.html` — an earlier, standalone HTML/CSS/JS mockup of the marketplace UI with a
  hardcoded sample catalog. Superseded by the real prototype in `frontend/` now, but kept around for
  reference since it's a nice snapshot of where this started.

## Not included in this package

The confidentiality/IP-assignment agreement (a legal document, unrelated to the actual codebase), the
presentation deck, and the WhatsApp demo video didn't make the cut here since they're not really part
of the technical project — but say the word if you'd like those folded in too.
