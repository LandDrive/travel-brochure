# Three Families, One Summer — destination brochure

Live: **https://landdrive.github.io/travel-brochure/**

One self-contained HTML file. No build step, no dependencies to install, no
server. Open `index.html` locally and it works; push to `main` and GitHub Pages
republishes it within a minute or two.

## Adding a destination

Everything on a tab is generated from one object in the `DESTINATIONS` array
near the top of the `<script>` block. Append one object following the shape of
the existing four and the tab bar, maps, photo cards, itinerary, budget table
and comparison rows all build themselves.

```js
{ id, tab, title, theme, lede, facts, hero,
  gmap, map, pinKinds, pins, doing, bases,
  itinerary, budget, alt, compare }
```

Checklist for a new one:

1. **`theme`** sets three CSS custom properties (`--dark`, `--accent`, `--sun`),
   which recolor the whole page when that tab is active. Pick a `dark` value
   visually distinct from the four in use:

   | Tab | `--dark` |
   |---|---|
   | Madeira | `#1D4237` green |
   | Greece | `#143845` blue |
   | Japan | `#1A2440` indigo |
   | Germany | `#2C3440` slate |

2. **`hero.type`** picks the SVG diagram at the top of the tab. Three exist:
   - `profile` — elevation cross-section (Madeira)
   - `hops` — linear route with travel times and costs (Greece, Japan)
   - `radius` — concentric drive-time rings (Germany)

   Adding a fourth means a new `heroX()` function plus a branch in
   `renderHero()`.

3. **Photo cards** — eight per destination, in `doing`. Each card resolves an
   image in this order:

   1. `photo:` — an explicit URL. **Always set this.**
   2. `wiki:` — live Wikipedia API lookup. Fallback only.
   3. `flickr:` — loremflickr keyword photo. Fallback only.
   4. A styled text placeholder.

   Do not rely on tier 2. See "Why the photos are hardcoded" below.

4. Commit and push. Pages redeploys from `main` root automatically.

## Why the photos are hardcoded

The `wiki:` lookup was the original mechanism and it is not reliable enough to
ship on:

- **Three of the 32 article titles have no lead image at all**, so those cards
  could never resolve (`Levada`, `Monte, Madeira`, `teamLab`).
- **Four more returned a map or a diagram rather than a photograph** — the
  `Knossos` article's lead image is a map of Crete, `Samaria Gorge` is likewise
  a map, `Shinkansen` is a route diagram, and `Santorini` is an administrative
  boundary map.
- It requires a network call at render time, which fails outright from
  `file://` (the browser sends `Origin: null` and Wikipedia rejects it).

Resolving each card to a permanent `upload.wikimedia.org` URL in `photo:` makes
tier 1 win, so the page needs no API call and renders identically from
`file://`, from Pages, and from anywhere else.

**When picking a replacement image, use the thumbnail URL exactly as the API
returns it.** Wikimedia only serves thumbnails at an allowlist of widths — hand
editing `/1280px-` down to `/800px-` returns HTTP 400.

## Getting images to resolve

Batch lookup, up to 50 titles at a time:

```
https://en.wikipedia.org/w/api.php?action=query&format=json&prop=pageimages&piprop=thumbnail&pithumbsize=1200&redirects=1&titles=Levada|Porto%20Moniz|Funchal
```

Read `thumbnail.source` off each page object, and match results back to cards
through the `normalized` and `redirects` arrays — the title you asked for is
often not the title you get back.

For a card the article lookup can't serve, search Commons directly:

```
https://commons.wikimedia.org/w/api.php?action=query&format=json&generator=search&gsrsearch=filetype:bitmap%20Knossos%20palace&gsrlimit=20&prop=imageinfo&iiprop=url&iiurlwidth=1280
```

## Things that are deliberate — don't "fix" them

- **The dark theme via `prefers-color-scheme` stays.** Without it, mobile
  browsers apply their own auto-darkening, which inverts the backgrounds while
  leaving the text dark and makes the page unreadable. This was a real bug and
  this is the fix.
- **The copy takes positions on purpose**, including where it contradicts the
  source research: the Japan cruise being the weaker option there, Greece's
  cruise being "the better deal and the worse trip," the two errors flagged in
  Germany's source cost table, and the argument that the JR Pass doesn't pay off
  post-2023 for that route. Don't neutralize these into both-sides copy.

## Open

- Whether the other two families are military households. It changes the
  Germany tab materially — Space-A carries only a sponsor and that sponsor's own
  dependents, so if two of three families are civilian, Germany's cost advantage
  largely goes away. Leave that tab alone until resolved.
