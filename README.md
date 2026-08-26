# Beechcraft

Daily aggregator of Beechcraft Model 18 "Twin Beech" (C18S, D18S, H18,
AT-11, C-45, SNB, etc.) and Model 17 "Staggerwing" classified listings
from [Barnstormers.com](https://www.barnstormers.com), published as a
static page (`docs/index.html`) meant to be embedded via `<iframe>` on
taildraggers.com.

Controller.com was evaluated (in the companion [Aeronca](https://github.com/taildraggers/aeronca)
repo) and dropped: its search results are only reachable through an internal
client-side widget (not a plain URL), which a headless browser can't drive
reliably for an unattended daily job.

Note: in the companion [Aviat](https://github.com/taildraggers/aviat),
[CubCrafters](https://github.com/taildraggers/cub-crafters),
[de Havilland](https://github.com/taildraggers/de-Havilland),
[Maule](https://github.com/taildraggers/maule),
[Van's RV](https://github.com/taildraggers/vans),
[RANS](https://github.com/taildraggers/rans),
[Luscombe](https://github.com/taildraggers/luscombe),
[Just Aircraft](https://github.com/taildraggers/just-aircraft),
[Kitfox](https://github.com/taildraggers/kitfox),
[Bellanca](https://github.com/taildraggers/bellanca),
[Stearman](https://github.com/taildraggers/stearman),
[Waco](https://github.com/taildraggers/waco),
[Pitts](https://github.com/taildraggers/pitts),
[Taylorcraft](https://github.com/taildraggers/taylorcraft), and
[Swift](https://github.com/taildraggers/swift) repos, Barnstormers'
single-manufacturer category pages turned out to include unrelated
listings mixed in with no distinguishing HTML markup. This repo is built
with the same fix from day one: `scraper/barnstormers.py` filters by
title against a small allowlist (see `TARGET_MODEL_PHRASES` in
`scraper/barnstormers.py`) before publishing.

This repo covers two source categories representing two very different
Beechcraft product lines - the **Model 18 "Twin Beech"** (a twin-engine
tailwheel transport, in production 1937-1969) and the **Model 17
"Staggerwing"** (a single-engine biplane with distinctive negative-stagger
wings). "Staggerwing" is a distinctive coined nickname with no ordinary-
English collision risk, so it's trusted standalone the same way
"Citabria" or "Kaydet" are elsewhere. Model 18 designations (`C18S`,
`D18S`, `H18`, and the military `AT-11`/`C-45`/`SNB` codes) are short and
generic-looking enough - especially the military ones, where a bare
"SNB" or "C-45" carries real collision risk - that every match requires
the title to also say "Beech" or "Beechcraft" explicitly (the same lesson
learned the hard way in the companion Piper repo, where a bare "Cub"
mislabeled non-Piper homebuilts as genuine Pipers). A bare "Beech"/
"Beechcraft" mention with no specific code is still published, labeled
`Twin Beech` since that's the only product line the Model-18 category
page covers - the same bare-brand-fallback policy used in the companion
Stearman/Waco/Pitts/Taylorcraft/Swift repos. Titles that read as parts,
accessories, services, or raffles are still dropped regardless.

**Gear note - this one's different from every taildragger-only make
covered so far.** The Model 18 has a *real, common* tricycle-gear
history: Volpar Inc. converted 323+ Model 18s to tricycle gear starting
in 1960, Beechcraft itself began offering factory tricycle gear as an
option in 1963, and Hamilton Aviation's "Westwind" conversions build on
Volpar's tricycle gear. These ads often don't use the word "tricycle" or
"nosewheel" anywhere - they just say "Volpar" or "Hamilton Westwind" - so
the standard text-based safety net used in every companion repo isn't
enough by itself. Any title or ad text matching "Volpar" or "Hamilton
Westwind" is **categorically excluded** on top of that safety net. The
factory H18 is *not* categorically excluded, since factory tricycle gear
was merely optional on it (buyer's choice) rather than a fixed model
attribute - individual tricycle-gear H18 ads are expected to be caught by
the general text-based safety net instead. The Staggerwing has no
tricycle-gear variant at all (like the companion Pitts/Waco repos), so no
Staggerwing-specific exclusion is needed.

## How it works

- `scraper/barnstormers.py` searches Barnstormers.com's Beech-18 and
  Staggerwing categories for listings, follows pagination, then keeps
  only the ones whose URL slug matches the allowlist (Barnstormers builds
  each listing's URL slug directly from the ad's own title, so this runs
  before any detail page is fetched). For the matches, it visits each
  listing's detail page to pull out the price, location, and posted date
  (falling back to regex heuristics over the visible text since the site
  doesn't expose structured data). The title is derived from the listing
  URL's own SEO slug, since every detail page shares one generic
  `<title>`/`<h1>`; the final parsed title is checked against the
  allowlist again as a safety net. Pagination is built directly from
  Barnstormers' known `?seocategory=<url-encoded-path>&page=<n>` URL
  pattern rather than discovered by following a "Next" link, since this
  category's pager renders as page-number buttons with no "Next" text or
  `rel="next"` attribute to find (a lesson learned the hard way in the
  companion Van's RV repo, where the link-following approach silently
  stopped after page 1).
- `main.py` runs the scraper, de-duplicates results, sorts them
  newest-posted-first, and renders them into `docs/index.html` titled
  **"Other Beechcraft Ads on the Web"**, with one row per listing: Title
  (linked to the original ad), Price, Location, Date Posted, and Site
  Posted On. Below phone width, each row collapses into a card (title +
  price on one line, location/date/site on a smaller line below) instead
  of a horizontally-scrolling table. Below the table, a "Search More
  Beechcraft Listings" section links out to Trade-A-Plane, Controller,
  and ASO - sites that block automated scraping, but are still worth
  sending visitors to directly via a pre-filled search. Links use
  `rel="noopener noreferrer"` and the page sets a `no-referrer` meta
  policy, so none of these sites see that the click came from
  taildraggers.com.
- `.github/workflows/daily-scrape.yml` runs the whole thing once a day (13:00 UTC),
  commits the regenerated `docs/index.html` if it changed, and can also be triggered
  manually from the Actions tab (`workflow_dispatch`).

## One-time setup: enable GitHub Pages

This repo publishes `docs/index.html` as a plain static file — GitHub Pages just needs
to be pointed at it once:

1. Go to **Settings → Pages** in this repository.
2. Under **Build and deployment → Source**, choose **Deploy from a branch**.
3. Branch: `main`, folder: `/docs`. Save.
4. GitHub will publish the page at `https://taildraggers.github.io/beech/`
   (may take a minute or two the first time).

Also check **Settings → Actions → General**:
- **Actions permissions**: "Allow all actions and reusable workflows".
- **Workflow permissions**: "Read and write permissions" (needed so the daily
  job can commit the regenerated page back to the repo).

## Embedding on taildraggers.com

```html
<iframe
  src="https://taildraggers.github.io/beech/"
  title="Other Beechcraft Ads on the Web"
  style="width: 100%; height: 800px; border: 0;"
  loading="lazy">
</iframe>
```

The page also posts its rendered height to the parent window on load/resize
(`{ type: "taildraggers:resize", height }`) so it can be auto-sized instead
of using a fixed guessed height - add a matching `message` listener on the
embedding page to pick this up.

## Running locally

```bash
pip install -r requirements.txt
playwright install --with-deps chromium
python main.py
```

This writes/overwrites `docs/index.html`.

## Notes

- If Barnstormers changes its markup or is briefly unreachable, the run logs will
  show a `[warn]`/`[error]` line pointing at what broke rather than failing silently.
- The scraper identifies itself with a browser-like `User-Agent` and adds a short
  delay between requests to be polite to the site.
- Two Barnstormers categories are configured
  (`category-16758-Beechcraft--Beech-18.html` and
  `category-16835-Beechcraft--Staggerwing.html`). If listings turn out to
  be split across additional categories, add more URLs to `CATEGORY_URLS`
  in `scraper/barnstormers.py`.
- The Model 18 military-designation list (`AT-11`, `C-45`/`UC-45`/`TC-45`,
  `SNB`) isn't exhaustive of every variant ever built. Missing a code
  isn't fatal since the bare-"Twin Beech" fallback still publishes the
  listing (just without a specific model in the title) - but if a
  particular code turns up often enough, add it to
  `scraper/barnstormers.py`.
