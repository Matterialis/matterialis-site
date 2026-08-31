# Matterialis landing page

Marketing site for **Matterialis, by Amphico** — a foundation model for
industrial materials. Live at **[matterialis.com](https://matterialis.com/)**.

One file, no build step: `index.html` contains all markup, CSS and JS. The
favicon is embedded as a data URI and the logo is an inline SVG symbol, so the
page works opened straight from disk.

```
index.html      the whole site
design.md       the design system: tokens, type, colour rules, voice
ANALYTICS.md    what the page measures and how to verify it
img/            use-case photography, credits, competitor and team logos
CNAME           matterialis.com
```

## Preview

```bash
python3 -m http.server 8000     # then open http://localhost:8000
```

Opening `index.html` from Finder also works — you only need the server for
`img/credits.js` to load.

Check it at 390px, 768px, 1440px and 1920px. The product mocks are drawn in
HTML/CSS at a fixed 1180px and scaled to their column. Below 1080px they are
deliberately **not** scaled: they keep the authored width inside a horizontal
scroller, because a width-fit scale at phone widths lands near 0.33 and the
tables stop being readable.

## Deployment

GitHub Pages, from `main` at the root. Push and it publishes; there is no build
step and nothing to sync. `.nojekyll` makes Pages serve the tree verbatim
rather than running it through Jekyll.

## Design

`design.md` is the whole design system in one file — every colour, type,
radius, motion and elevation token with its value, plus the rules that carry
the look. Read it before changing anything visual.

The tokens are **copied into** `:root` in `index.html` rather than imported,
which is what keeps the page portable. That makes a token change a two-file
edit, and nothing enforces the match.

The two rules most often broken by accident: the **accent** colour marks only
the active state and what the AI contributed, and the **evidence** colours
(green / amber / red) report only how well supported a claim is. Neither is
ever decoration.

## Photography

Six use-case cards, greyscale photos in `img/`:

```
defence.jpg  membranes.jpg  compounding.jpg  cosmetics.jpg  food.jpg  pharma.jpg
```

Machined parts on engineering drawings, a fine technical weave, polymer
granulate, gel samples, a powder blend, and a sample vial in a rack. All
greyscale, 1000×563, under 90KB, **no people in frame**. `img/credits.js`
carries the Unsplash attribution the page renders under the cards.

## Analytics

The page reports to PostHog (EU): reading depth per section, exit points,
use-case and FAQ interest, and CTA clicks. See **`ANALYTICS.md`** for the event
list, the queries to start from, the opt-out URLs, and the one change still
outstanding — pointing ingestion at a first-party proxy.

Nothing about the analytics is load-bearing for the page: both blocks are
self-contained, and deleting them leaves the site behaving identically.
