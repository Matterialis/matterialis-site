# Analytics — how PostHog is set up

Everything the landing page records, where it goes, and how to check it works.

| | |
|---|---|
| Product | PostHog Cloud **EU** |
| Project | **Matterialis**, id `105300` |
| Project token | `phc_GnUG6LSAkfO92r5vN5asriOtooGLbGPx3V40gANPcBi` |
| Ingestion host | `https://eu.i.posthog.com` — not yet proxied, see [Pending](#pending-the-first-party-proxy) |
| Code | `index.html`, two blocks marked `ANALYTICS` |

The project token is **not a secret**. It is a write-only public key, it is
already in the HTML served to every visitor, and it cannot read data back.
Don't treat it as a credential or try to rotate it if it "leaks".

## What is in the page

Two fenced blocks, both greppable as `ANALYTICS`:

**In `<head>`** — the `MTL_ANALYTICS` config object, the posthog-js loader, and
identity: visit counting, the `?c=` campaign parameter, opt-out handling, the
manual `$pageview`, and the human gate that decides when a visit becomes real.

**Before `</body>`** — the instrumentation: the engaged-time clock, click and
FAQ listeners, and the exit event.

The blocks are self-contained. They read the DOM and listen; they do not hook
`open()`, the FAQ handler, or anything else in the page's own script. **Delete
both and the page behaves identically.** Everything reports through
`window.mtl.track()`, which silently no-ops if posthog is missing, blocked, or
opted out — the page must never break because analytics did.

## The scanner gate

Email security scanners (Defender Safe Links, Mimecast, Proofpoint) prefetch
every URL in every message they scan. They render the page headlessly and
leave. At cold-outreach volume they outnumber real visitors, so nothing is
attributed and **no session replay starts** until a human signal arrives:
pointer, key, touch, wheel, scroll, or three seconds of visible time.

`$pageview` still fires on every arrival, which is the point — it keeps the
scanner share measurable as `$pageview` minus `visit_engaged`, instead of
silently inflating the real numbers. Don't "simplify" that gate away.

## What gets recorded

Custom events, on top of PostHog's own `$pageview` / `$pageleave` / autocapture:

| Event | Fires when | Key properties |
|---|---|---|
| `visit_engaged` | a human signal arrives (see the gate above) | `from_outreach`, `visit_number` |
| `section_read` | a section passes 10s of engaged time (once each) | `section`, `engaged_seconds` |
| `product_claim_read` | one of the four product rows passes 8s (once each) | `claim`, `title` |
| `use_case_opened` | a use-case card is opened | `case` |
| `use_case_closed` | the inspector closes, however it closed | `case`, `seconds_open` |
| `faq_opened` | an FAQ is expanded | `question` |
| `cta_click` | a booking or mailto link | `kind`, `location`, `label` |
| `nav_click` | any in-page anchor | `to`, `location` |
| `compare_scrolled` | the compare table is scrolled sideways | — |
| `page_exit` | the tab is hidden or the page unloads | `last_section`, `section_path`, `read_*` per section, `max_depth_pct`, `total_engaged_seconds` |

On **every** event: `visit_number`, `is_returning`, `days_since_first_visit`,
and a campaign token when the visit came from a `?c=` link.

Two things to know about the numbers:

- **Engaged time, not time on page.** Whichever section holds the viewport
  centre owns the clock, so the totals sum instead of double-counting overlaps.
  The clock stops on tab blur and after 30s of no scroll, pointer or key, and
  per-tick deltas are capped — a laptop waking from sleep cannot register as a
  long careful read.
- **`section_read` fires live; `page_exit` is best-effort.** Unload delivery is
  never guaranteed, so the strongest signal (a section genuinely read) is sent
  at the moment it happens. `page_exit` goes out via `sendBeacon` on
  `visibilitychange`, never `unload` — `unload` and `pagehide` drop silently on
  mobile Safari.

`cta_click` identifies a booking link by matching the booking host. **Change
the booking provider and you must add its host to that test**, or every demo
click stops being recorded, with no error and nothing to notice until the
funnel looks empty.

## Where to look

| | |
|---|---|
| Visitors, referrers, countries, devices | [Web analytics](https://eu.posthog.com/project/105300/web) |
| Watch real sessions | [Session replay](https://eu.posthog.com/project/105300/replay/home) |
| Raw event stream, live | [Activity](https://eu.posthog.com/project/105300/activity/explore) |
| Clicks and scroll depth | [Heatmaps](https://eu.posthog.com/project/105300/heatmaps) |
| Ad-hoc questions | [SQL editor](https://eu.posthog.com/project/105300/sql) |
| Settings | [Project](https://eu.posthog.com/project/105300/settings/project) · [Replay](https://eu.posthog.com/project/105300/replay/settings) |

## Queries to start from

All three parse and run as written.

**Which sections actually get read** — average engaged seconds per section:

```sql
SELECT round(avg(toFloat(properties.read_hero)), 1)    AS hero,
       round(avg(toFloat(properties.read_product)), 1) AS product,
       round(avg(toFloat(properties.read_cases)), 1)   AS cases,
       round(avg(toFloat(properties.read_compare)), 1) AS compare_,
       round(avg(toFloat(properties.read_trust)), 1)   AS trust,
       round(avg(toFloat(properties.read_faq)), 1)     AS faq,
       round(avg(toFloat(properties.read_cta)), 1)     AS cta
FROM events
WHERE event = 'page_exit' AND timestamp >= now() - INTERVAL 30 DAY
```

**Where people leave:**

```sql
SELECT properties.last_section AS exited_on,
       count() AS visits,
       round(avg(toFloat(properties.total_engaged_seconds))) AS avg_seconds,
       round(avg(toFloat(properties.max_depth_pct)))         AS avg_depth_pct
FROM events
WHERE event = 'page_exit' AND timestamp >= now() - INTERVAL 30 DAY
GROUP BY exited_on
ORDER BY visits DESC
```

**How much of the traffic is email security scanners** — the gap between
arriving and doing anything. Expect this to be large; measuring it is the whole
reason the gate exists:

```sql
SELECT countIf(event = '$pageview')      AS arrivals,
       countIf(event = 'visit_engaged')  AS human_visits,
       round(100 * countIf(event = 'visit_engaged')
             / nullif(countIf(event = '$pageview'), 0), 1) AS pct_human
FROM events
WHERE event IN ('$pageview', 'visit_engaged')
  AND timestamp >= now() - INTERVAL 30 DAY
```

## Pending: the first-party proxy

**Do this before any real campaign.** Ingestion currently goes straight to
`eu.i.posthog.com`. Corporate proxies at large industrials blocklist
`*.posthog.com` wholesale, so an estimated 20–40% of events never arrive — and
the loss is biased toward exactly the enterprise visitors this page exists for,
which is worse than a random gap.

The fix is DNS only, no build step:

1. Create the proxy record in PostHog ([reverse proxy
   settings](https://eu.posthog.com/project/105300/settings/organization-proxy);
   0 of 2 records used). It returns a CNAME target.
2. Add `CNAME e.matterialis.com -> <that target>`. PostHog verifies and issues
   the certificate automatically.
3. In `index.html`, swap the two adjacent `host:` lines in the `MTL_ANALYTICS`
   config. Nothing else changes — the loader fetches its static assets from
   whichever host is set.

## Checking it works

Load the page, open devtools → Network, filter `posthog`. You want a `200` on
`array.js` and a `POST .../i/v0/e/`.

**The one trap:** posthog-js drops all its own events when
`navigator.webdriver` is true. Any headless or automated check of this page will
look completely broken and is not. To test through automation:

```js
posthog.set_config({ opt_out_useragent_filter: true })
```

That is PostHog's own bot filter working correctly, and it sits behind the
scanner gate rather than being the only defence.

If events still do not arrive: check you are not opted out
(`posthog.has_opted_out_capturing()`), and remember `localhost` traffic is
excluded from insights by a test-account filter — it is ingested, just filtered
out of charts by default.

## Privacy posture and opt-outs

There is no consent banner. What is in place instead:

- Data is hosted in the **EU**.
- Session replay **masks every input**.
- `recording_domains` restricts replay and heatmaps to our own origin, so a
  copied token cannot record anywhere else.
- Console log capture is off.

| URL | Effect |
|---|---|
| `matterialis.com/#do-not-track` | opts you out permanently |
| `matterialis.com/#internal` | the same — use once on every team machine, or at this volume we outnumber real visitors |
| `matterialis.com/#track-me` | reverses either |

## Configured settings

For audit or restore. Set over the PostHog API on 25 Aug 2026:

| Setting | Value |
|---|---|
| Timezone / week start | `Europe/London`, Monday |
| Session replay | on, 100% sample, **3000ms minimum duration** |
| Replay retention | `30d` — plan maximum |
| Heatmaps, dead clicks, web vitals | on |
| Console log capture | off (noise on a static page) |
| `recording_domains` | the two matterialis.com origins |
| `data_attributes` | `data-attr`, `data-case`, `data-s` |
| Test-account filters | excludes `localhost` and the legacy `jordiferrero.github.io` host |

The 3000ms minimum duration is what keeps scanner hits out of the replay list.

## Known limits

- **Replay retention is 30 days**, the plan maximum — a longer window was
  rejected by the API. Events keep 12 months, so the behavioural trail outlives
  the video: if someone replies six weeks later the numbers are still there,
  the recording is not.
- **The project holds ~1,900 dormant events from a previous product** on
  `jordiferrero.github.io` (Dec 2025 – Apr 2026, nothing since). PostHog has no
  API to bulk-delete ingested events, so they are excluded by a `$host`
  test-account filter instead. Leave "filter test accounts" on for all-time
  views or they reappear.
