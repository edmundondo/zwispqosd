# Changelog

All notable changes to the Zimbabwe ISP Tracker are recorded here. Format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/); versioning follows
[Semantic Versioning](https://semver.org/) (MAJOR.MINOR.PATCH).

The version number shown here matches the `<meta name="app-version">` tag in `index.html`
and the `v{version}` badge in each page's footer.

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
