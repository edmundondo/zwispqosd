# Changelog

All notable changes to the Zimbabwe ISP Tracker are recorded here. Format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/); versioning follows
[Semantic Versioning](https://semver.org/) (MAJOR.MINOR.PATCH).

The version number shown here matches the `<meta name="app-version">` tag in `index.html`
and the `v{version}` badge in each page's footer.

## [1.2.0] — 2026-09-10

### Removed
- **All export options (Export PDF Report, Export ISP CSV, Export QoS CSV, Export Status CSV,
  Export Speed CSV, Export EPUB) have moved to the privileged `zwispqosp` admin app.** This demo
  is the shared public-access surface — exporting the underlying data is now part of the
  privileged/analytics tier, matching the split Ed set out: `zwispqosd` = public demo, `zwispqosp`
  = privileged backend for analytics, reporting, tweaking, and upgrading. Removed the six toolbar
  buttons, their click handlers, the CSV/EPUB builder functions (`downloadCsv`, `escapeXml`,
  `xhtmlWrap`, `buildEpub`), the now-unused `btn_export_*` translation keys in every language
  block, and the JSZip `<script>` tag (nothing on this page uses it anymore).
- The in-page free "sample ISP benchmark report" teaser (the one whose call-to-action pitches a
  licensed subscription) is untouched — that's a different feature from the download buttons and
  stays here as the public-facing teaser it's meant to be.

### Notes
- The equivalent exports in `zwispqosp` are actually more capable than the ones removed here:
  they read Supabase's full history for the selected site rather than this page's client-side
  1000-row cache, and they cover every site the admin app knows about via its site selector, not
  just Zimbabwe.

## [1.1.4] — 2026-09-09

### Fixed
- The v1.1.3 fix improved the *messaging* around automatic-speed-test failures but didn't fix an
  underlying bug that had been there since v1.1.0: the CDN import used
  `import { SpeedTest } from ".../+esm"`, but jsDelivr's `/+esm` transform for this package exports
  the class as the module's `default` export, not a named `SpeedTest` export — confirmed live by
  checking `Object.keys(mod)`, which is just `["default"]`. The named import was always `undefined`,
  so `window.__CFSpeedTest` never actually got set even when the network fetch itself succeeded
  perfectly, and every real click landed on "Couldn't load the automatic speed test (network or
  ad-blocker issue)" — a real, accurate-sounding error message pointing at completely the wrong
  cause. Now accepts either `mod.default` or `mod.SpeedTest`, and treats a missing class as a
  genuine failure rather than a false success. Verified live against the real jsDelivr CDN
  (instantiated the corrected class successfully; did not run a full test to avoid using data).

## [1.1.3] — 2026-09-09

### Fixed
- Investigated a report that the automatic speed test's "Automatic testing isn't supported in
  this browser" message was appearing identically in Chrome, Edge, Firefox, and Brave — all four
  fully support the `fetch`/`Promise`/`Worker` APIs the code checks for, which ruled out an actual
  capability gap. The real cause: `runAutoSpeedTest()` treated `!window.__CFSpeedTest` at click
  time as "this browser can't do it," but `window.__CFSpeedTest` is only set once the
  `@cloudflare/speedtest` library finishes an async CDN import (a `<script type="module">` fetch
  that isn't synchronized with the rest of the page's script at all) — so any click that landed
  before that fetch resolved, or any click when the fetch failed outright (CDN blocked, offline,
  ad-blocker), hit the same wrong, misleading "not supported" text regardless of browser.
  `runAutoSpeedTest()` now awaits a `window.__CFSpeedTestReady` promise the module script exposes,
  showing a distinct "Loading the speed test tool…" state while the import is in flight and a
  distinct "Couldn't load the automatic speed test (network or ad-blocker issue)" message only if
  it genuinely fails.

## [1.1.2] — 2026-09-01

### Fixed
- `.row-top` (the rank/name/subscriber-count/QoS-badge/status-pill/caret strip on each provider row
  in `index.html`) was a `flex-nowrap` row with several fixed-width children totaling ~370px+ —
  it never fit under a 320px viewport, forcing horizontal scroll on small phones. Added a
  `@media (max-width:480px)` rule that lets `.row-top` wrap: name takes the full first line,
  subscriber count / QoS badge / status pill / caret wrap to a second line, left-aligned.

## [1.1.1] — 2026-08-24

### Changed
- Clickable provider rows on `index.html` now carry a light green border (`rgba(62,207,142,.35)`)
  to visually mark that each row has a drill-down (expand) behind it, distinct from static
  content. Once opened, the border goes solid bold green (`var(--green)`, 2px) so "clickable but
  closed" and "currently open" read as two clearly different states rather than one flat hover
  color change.

## [1.1.0] — 2026-08-23

### Added
- Built-in automatic speed test on `index.html`: a "▶ Run automatic test" button next to each
  provider's manual speed-report form runs a free, open-source client-side test
  ([`@cloudflare/speedtest`](https://github.com/cloudflare/speedtest), BSD-3-Clause, no signup or
  API key) and auto-fills the download/upload fields — manual entry stays available for anyone
  who prefers it or whose browser can't run the automatic test.
- Richer speed-report schema: alongside download/upload Mbps, an automatic test also captures
  idle/download/upload ping, jitter, and packet loss, plus a best-effort connection type and
  device type (no IP-based geolocation or ISP-name lookup is performed — that data is
  deliberately not collected, consistent with this site's existing privacy design).
- CSV export for speed reports now includes the new columns (Method, pings, jitter, packet loss,
  connection, device) alongside the original ones.
- Supabase `speed_reports` table migrated (see `matokipedo-isp-tracker-qc`'s SKILL.md) to store
  the richer columns; the site still degrades gracefully to the original base columns if a
  Supabase project hasn't been migrated yet.

### Notes
- The automatic test needs `fetch`, `Promise`, and Web Workers, all of which the Lite page's
  compatibility banner already checks for — anyone routed to Lite was never going to see this
  button anyway, and on `index.html` itself the button hides itself with a plain-language note
  if the browser can't run it.
- Data usage for the automatic test is intentionally light — a few MB, not the tens of MB a full
  speed test can use — to stay considerate of visitors on metered mobile data.

## [1.0.0] — 2026-08-22

Baseline versioned release.

### Added
- `lite.html` — no-script fallback page for older phones / slow connections, listing all 10
  POTRAZ-tracked providers with an email- and SMS-based rating option (no backend required).
- Language switching on the Lite page: `lite.html` (English) plus `lite-sn.html` (ChiShona),
  `lite-nd.html` (IsiNdebele), `lite-ny.html` (Chewa), `lite-ts.html` (Shangani/Xitsonga),
  `lite-st.html` (Sotho), `lite-tn.html` (Tswana), `lite-ve.html` (Venda), `lite-xh.html` (Xhosa).
  Draft, machine-assisted translations, flagged as unreviewed on the page itself.
- Cross-browser compatibility banner on `index.html`: feature-detects `fetch`, `Promise`,
  `localStorage`, and CSS grid/flexbox support; points visitors to the Lite page if anything's
  missing, rather than failing silently.
- Version tracking: `<meta name="app-version">` on every page, a footer version badge, and this
  changelog.

### Fixed
- `matokipedo-logo.jpg` was actually HEIC-encoded photo data saved with a `.jpg` extension (an
  iPhone Photos/Preview export quirk) — every browser failed to decode it as the JPEG its
  extension and Content-Type header claimed, showing a broken-image icon everywhere. Re-encoded
  as a genuine JPEG, same filename and dimensions.

### Not yet translated
TjiKalanga, Chibarwe, Khoisan (Tjwao), Nambya, Ndau, and Tonga are not yet available as Lite-page
translations — see `SKILL.md` in the `matokipedo-isp-tracker-qc` skill for why (low confidence in
producing an honest draft rather than a fabricated one for these lower-resourced languages).
They're listed but greyed out in the Lite page's language bar until a real translation exists.

<!--
Template for the next entry — copy this when you ship a change:

## [1.1.0] — YYYY-MM-DD
### Added / Changed / Fixed / Removed
- ...
-->
