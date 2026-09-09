# Matterialis — design

The design system for this site, in one file. Everything the page needs is
here; nothing lives in a parent folder any more. If you are changing how the
page *looks*, this is the document to read first.

The tokens below are **copied into** `:root` at the top of `index.html` rather
than imported, which is what keeps the page a single portable file. Change a
value here and you must change it there too — nothing enforces it.

---

## The one-line brief

**An instrument, not a SaaS product.** A power tool for scientists: calm,
precise, evidence-led. The design should read as credible to a chemist and
boring to a growth marketer.

It should feel like scientific instrumentation, a modern materials lab, or
high-quality engineering software. Restrained and spacious, with complexity
presented as organised rather than hidden.

**Avoid:** corporate enterprise clichés, consumer-app friendliness, flashy AI
gradients, dense legacy scientific software, and any decoration that does not
carry meaning.

---

## Colour

Four families, and each has one job. The discipline that makes the system work
is that **two of them are reserved** and must never be borrowed for emphasis.

### Chrome — dark surfaces

The app bar, the nav, anything that frames the work rather than being the work.

| Token | Hex | Use |
|---|---|---|
| `--chrome` | `#141618` | base chrome surface |
| `--chrome-raised` | `#1E2022` | raised within chrome |
| `--chrome-line` | `#2A2C2E` | rules on chrome |
| `--chrome-fg` | `#FFFFFF` | primary text on chrome — 18.14:1 |
| `--chrome-fg-strong` | `#D5D9DC` | emphasis |
| `--chrome-fg-muted` | `#AEB5BA` | secondary — 8.74:1 |
| `--chrome-fg-dim` | `#8A9297` | tertiary — 5.73:1 |

The chrome is this dark specifically so the white logo lands at 18.14:1.

### Paper, ink and rules — the working surface

| Token | Hex | Use |
|---|---|---|
| `--paper` | `#FDFDFC` | page |
| `--paper-raised` | `#FFFFFF` | grid, card |
| `--paper-strip` | `#FBFBFA` | header, filter strip |
| `--paper-sunken` | `#F5F6F6` | filled field |
| `--rule` | `#E8EAEC` | zone separator |
| `--rule-soft` | `#F2F3F4` | row separator |
| `--ink` | `#17191B` | body text |
| `--ink-muted` | `#6C7276` | secondary text |
| `--ink-faint` | `#9CA1A5` | tertiary text |

**Hairline rules do the structural work.** Cards are for summaries only;
anything tabular gets a rule-separated grid, not a card.

### Accent — reserved

| Token | Hex | Use |
|---|---|---|
| `--brand-signal` | `#0E6B72` | active state, focus, the reasoning rail |
| `--brand-signal-tint` | `#F2F7F7` | selected row fill |
| `--brand-signal-line` | `#CFE2E3` | line on tint |

The accent marks exactly two things: **where you are** (active nav, selected
row, focus) and **what the AI contributed**. Never decoration, and never as a
fill behind body text.

### Evidence — reserved

| Token | Hex | Meaning |
|---|---|---|
| `--evidence-strong` | `#186B4F` | corroborated / meets the criterion |
| `--evidence-limited` | `#8F6209` | single source / partial |
| `--evidence-weak` | `#A63A29` | contradicted / misses |
| `--evidence-neutral` | `#6C7276` | unassessed |

These report **how well supported a claim is, and nothing else**. Never a
category, never a brand accent, never decoration. This is the rule most likely
to be broken by accident, and breaking it costs the page its credibility.

### Data ramp — neutral by design

`--chart-1 #141618` · `--chart-2 #4A5054` · `--chart-3 #7C8287` ·
`--chart-4 #AFB4B8` · `--chart-5 #D8DBDD`

Data never borrows the accent. The single exception on this page is the query
grade in the similarity map, which is allowed because it is the active
selection.

### Logo blue — never in the interface

The logo stripe is **flat `#3678F3`**, and that value exists only inside the
logo artwork. Saturated blue on white is the signature of a generic AI product,
which is precisely why this system's accent is teal. The stripe was a
`#204CBF -> #3678F3` gradient until 9 Sep 2026; it is now the single bright
stop, so **there is no legal gradient anywhere in the system**, the logo
included. The masters live in `img/brand/` (lockup with the "by Amphico"
endorsement, wordmark, mark, each in colour and reverse). The lockup with the
in-artwork "by Amphico" endorsement is for closing/anchor placements — the
site footer, the deck cover and closing slide. Running chrome stays plain: the
site nav and the in-app mock bars use the wordmark-only variant, no
endorsement.

---

## Type

**Geist** and **Geist Mono**, currently loaded from Google Fonts.

```css
--font-sans: Geist, ui-sans-serif, system-ui, -apple-system, "Segoe UI", sans-serif;
--font-mono: "Geist Mono", ui-monospace, SFMono-Regular, Menlo, monospace;
```

### Scale

Small by consumer-web standards, on purpose — this is instrument density.

| Token | Size | Use |
|---|---|---|
| `--text-micro` | 9.5px | micro-labels |
| `--text-2xs` | 10.5px | dense metadata |
| `--text-xs` | 11.5px | status labels, hints, metadata |
| `--text-sm` | 12px | compact UI |
| `--text-base` | 12.5px | **body default** — grid cells, nav labels |
| `--text-md` | 13px | emphasis body |
| `--text-lg` | 14px | small headings |
| `--text-xl` | 15px | headings |

Weights: 400 regular, 450 book, 500 medium, 550 strong, 600 semibold.

Tracking: `--tracking-title -.022em`, `--tracking-name -.015em`,
`--tracking-figure -.03em`, `--tracking-micro .09em`.

### The wordmark is not a font

The logo is **a custom geometric grotesk drawn as outlines**. It is not Geist,
and it is not any typeface you can set. **Always place the logo artwork; never
re-set the wordmark in type.** The SVGs carry no `<text>` and no font metadata,
so there is nothing to fall back to if you try.

Geist was chosen for the UI partly because its flat terminals and tight fit
track the wordmark, which is why the two sit together well. That resemblance is
deliberate and is not permission to substitute one for the other.

On this page the logo is an inline SVG symbol (`#mtl-logo`), so there is no file
to lose. Consequence worth knowing: the second "t" in Matterialis is a **derived
glyph**, made by duplicating the existing "t" outline and shifting "erialis"
right, because there was no font to retype the name in. It still deserves a
designer's eye — a real drawing would kern or ligature the `tt` pair rather than
repeat the glyph.

### Two type rules that are the signature

**Micro-labels** (`.micro`) are mono, uppercase and tracked. They label zones,
groups and columns — **never body copy**. Write them in sentence case and let
CSS render them uppercase.

**Numbers are always mono with tabular figures on**, so decimal columns align
down a table. Never proportional figures in a table.

---

## Shape

| Token | Radius | Use |
|---|---|---|
| `--radius-indicator` | 2px | dots, meters, badges |
| `--radius-control` | 3px | buttons, inputs, nav |
| `--radius-surface` | 6px | panels, cards |
| `--radius-frame` | 10px | window, dialog |
| pill | 9999px | switches only |

The reasoning is worth keeping: 0px reads as a technical drawing and 6–8px
reads as a generic framework component. **2–3px is the radius of machined and
moulded objects** — instrument keys, switch caps, equipment panels.

---

## Elevation

```css
--shadow-menu:    0 4px 10px -2px rgb(20 22 24 / .10), 0 1px 3px rgb(20 22 24 / .06);
--shadow-overlay: 0 20px 44px -14px rgb(20 22 24 / .20);
--shadow-frame:   0 1px 2px rgb(20 22 24 / .04), 0 12px 32px -16px rgb(20 22 24 / .15);
```

**Working surfaces carry no shadow at all.** The grid, filter panel and
inspector are separated by 1px rules. Shadow is for menus and overlays only.
No inner shadows anywhere.

---

## Motion

| Duration | Token | Use |
|---|---|---|
| 75ms | `--dur-instant` | menu item highlight, row hover |
| 130ms | `--dur-fast` | **the default** — hover, focus, tab switch |
| 180ms | `--dur-medium` | dialog enter (4px rise, no scale) |
| 260ms | `--dur-slow` | meter fill, skeleton pulse |

Easing: `--ease cubic-bezier(.4,0,.2,1)` standard,
`--ease-out cubic-bezier(0,0,.2,1)` for entrances.

Everything respects `prefers-reduced-motion`.

---

## States

Focus is a **1px hairline**, not a glow.

> A translucent bloom around a focused field was the strongest amateur signal
> in the previous system. Precision instruments use a hairline.

No press-state transform. Hover is a surface change, not a lift.

---

## Layout

The page wraps at `--wrap 1240px` with `--gutter 32px`.

The app mocks are drawn in HTML/CSS at a fixed 1180px inside `.scaler` and
transform-scaled to their column. Below 1080px they are **not** scaled — they
keep the authored width inside a horizontal scroller, because a width-fit scale
at phone widths lands near 0.33 and the tables become unreadable.

For reference, the app's own structural sizes: chrome 46px, grid row 40px,
header row 29px, cell padding 7px, nav rail 166px, filter panel 196px,
inspector 210px. Control heights 25 / 26 / 30 / 34px.

---

## Voice

Clear, concise, expert. A trusted technical assistant: direct, evidence-led,
calm.

**Do**

- "Recommended based on similar thermal behaviour at equivalent HLB."
- "Holds viscosity within 8% of the incumbent, on limited evidence."
- "Closer thermal match than the incumbent, but a narrower supply base."

**Don't**

- "Revolutionary AI finds your perfect material!"
- "Supercharge your workflow 🚀"
- "Magic formulation intelligence, unlocked."

### Tells to keep out of marketing copy

These creep back in on every edit:

- **Negative parallelism.** "X, not Y" and "not just X, it's Y" are the single
  most common AI signature in marketing copy.
- **Em dashes.** The use-case section is at zero and should stay there. A
  comma, a colon or a full stop does the same work without the tell.
- **Forced triads.** Three real, concrete things is fine. Three abstract nouns
  in a row is not.

Prefer plain business language over anything that sounds like positioning.

---

## Imagery

Macro material textures, lab surfaces, scientific instruments, clean material
samples. Greyscale.

**No people in frame.** Avoid generic AI brain imagery, smiling-scientist stock,
futuristic holograms, dark cinematic lab scenes, and plastic-looking 3D
molecules.

---

## Marking AI-generated values

Wherever a model-generated value appears next to a measured one it must be
marked — `<sup class="ai">AI</sup>` inline, or the `badge-ai` pill.

This is a trust requirement, not a style choice. It is the whole reason the
evidence colours mean anything.
