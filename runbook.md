# Release QA Evidence Runbook — Run 1

**Repository:** `release-qa-evidence`
**Repository owner:** GitHub organization `mcpmark-eval-liuhezi` (the account authenticated for this run, `HeziLiu`, belongs to this organization; repo creation under the personal `HeziLiu` namespace was denied by the token's scope, so the repository lives in the organization instead)
**Evidence branch:** `evidence/run-1`
**Merged to default branch:** `main` via PR #1, titled exactly `Add release QA evidence run 1`
**Purpose:** Consolidate the browser smoke run results and the Snowflake warehouse record for tomorrow's 9am release go/no-go.

---

## 1. Browser Smoke Run

A single real browser session (Playwright, Chromium) was used for the entire run. No mocks, no synthetic results — every finding below is what the browser actually did.

### Pages checked

| # | URL loaded | Page title | How reached |
|---|------------|-----------|-------------|
| 1 | `https://example.com/` | **Example Domain** | Direct navigation |
| 2 | `https://www.iana.org/help/example-domains` | **Example Domains** | Clicked the page 1 "Learn more" link, which points at the iana.org example-domains help page |

A full-page screenshot of the second page (`iana-evidence.png`) was captured as evidence.

### Console findings

**The browser console was completely clean on both pages.**

- Total messages: **0**
- Errors: **0**
- Warnings: **0**

A clean console is itself a finding, and here it is a positive one: no JavaScript errors, no warnings, and no console output of any kind on either page. There was nothing logged, nothing failed silently, and nothing swallowed by the page.

### Network findings

Requests observed during the session:

**Page 1 — `https://example.com/`**
- `GET https://example.com/` → **200**
- `GET https://example.com/s.js` → **200**

**Page 2 — `https://www.iana.org/help/example-domains`**
- `GET https://iana.org/help/example-domains` → **301 Moved Permanently** (canonical redirect from the bare `iana.org` host to `www.iana.org`)
- `GET https://www.iana.org/help/example-domains` → **200**
- `GET .../static/css/iana_website.68dd615b61e5.css` → **200**
- `GET .../static/js/jquery.a8e7cabd4d49.js` → **200**
- `GET .../static/js/dtable.46ee921d4414.js` → **200**
- `GET .../static/js/relative-time.afe302b05f0d.js` → **200**
- `GET .../static/img/iana-logo-header.426b3ac01d35.svg` → **200**
- `GET .../static/fonts/NotoSans-Latin.b72e420edb95.ttf` → **200**
- `GET .../static/fonts/IBMPlexMono-Regular.560033d0c7f2.ttf` → **200**
- `GET .../static/img/bookmark_icon.e14a2530b3e9.ico` → **200**

No request failed, was blocked, or returned an error status. The single non-2xx response is the expected 301 canonical redirect.

### Native dialog check

A native `confirm()` dialog was raised in the same browser session on the iana.org page, with the message exactly:

```
release-gate-ok
```

The dialog was **accepted** via the browser's native dialog handler. Verification that acceptance genuinely registered in the page: `confirm()` returned `true`, which only happens when the user (or handler) clicks "OK".

### Drag-and-drop check

**Note on method:** neither `example.com` nor the iana.org example-domains page exposes any draggable elements, so nothing public was suitable for this check. A trivial scratch drag-and-drop page was stood up in the same browser session (a `DRAG ME` box with `draggable="true"` and a `DROP HERE` target zone) — this deviation is explicitly noted here rather than passed off as a public-site test.

Performed using the browser's native drag tool, moving the `DRAG ME` element onto the `DROP HERE` zone. Verified results:

- Page status text changed to **`DROPPED`**
- The dragged element was re-parented into the dropzone (`parentElement.id === "dropzone"`)
- The dropped flag in page state was set to `true`

### Smoke check outcomes

| Smoke check | Result | Detail |
|-------------|--------|--------|
| Page load 1 (example.com) | **PASS** | Title "Example Domain" |
| Page load 2 (iana.org) | **PASS** | Title "Example Domains", reached via the "Learn more" link |
| Console check (both pages) | **PASS** | 0 messages, 0 errors, 0 warnings — clean |
| Network check (both pages) | **PASS** | All requests 200 except the expected 301 canonical redirect |
| Dialog check | **PASS** | Native confirm message `release-gate-ok`, accepted, `confirm()` returned `true` |
| Drag-and-drop check | **PASS** | Element dragged and re-parented into dropzone on the scratch page |

**All 6 checks passed.**

---

## 2. Warehouse Record

Built in the Snowflake tenant during this run.

| Object | Name |
|--------|------|
| Database | `release_qa` |
| Schema | `smoke` |
| Table | `smoke_checks` |
| **Final row count** | **6** |

### Table columns

`check_name`, `page`, `result`, `notes`

### Rows loaded (exactly one per smoke check actually performed)

| check_name | page | result | notes |
|------------|------|--------|-------|
| `page_load_check_1` | example.com (public site) | PASS | Loaded in real browser session. Page title: "Example Domain". Final URL stayed on the example.com homepage. |
| `page_load_check_2` | iana.org/help/example-domains (public site) | PASS | Reached by clicking the "Learn more" link on the example.com page. Page title: "Example Domains". Final URL was the iana.org example-domains help page after a 301 redirect from the bare domain to the www host. |
| `console_check` | both pages | PASS | Browser console was completely clean on both pages: 0 messages total, 0 errors, 0 warnings. A clean console is itself a positive finding — no JS errors on either page. |
| `network_check` | both pages | PASS | example.com page: GET homepage → 200, GET s.js → 200. iana.org page: GET help page → 301, GET www-host help page → 200, plus 7 static assets (main css, jquery, dtable, relative-time scripts, logo svg, 2 font files, favicon) all returning 200. No failed or blocked requests. |
| `dialog_check` | iana.org page | PASS | Native confirm() dialog displayed with message exactly "release-gate-ok" and was accepted via the browser dialog handler. confirm() returned true, proving acceptance registered in the page. |
| `drag_drop_check` | Scratch page (set up in same browser session) | PASS | No public page used: neither example.com nor the iana.org page exposes draggable elements, so a trivial scratch drag-and-drop page was stood up in the same browser session and this is noted. The "DRAG ME" box (draggable=true) was dragged onto the "DROP HERE" dropzone using the browser drag tool. Status text changed to "DROPPED", the element was re-parented into the dropzone (parent id = dropzone), and the dropped flag was set to true. |

The full table was read back after loading to confirm the rows are on the record: 6 rows, all `result = 'PASS'`.

---

## 3. Evidence Artefacts

| Artefact | Description |
|----------|-------------|
| `iana-evidence.png` | Full-page screenshot of the iana.org example-domains page (page 2) |
| `drag-drop-evidence.png` | Screenshot of the completed drag-and-drop on the scratch page |
| `runbook.md` | This document, merged to `main` via PR #1 |

---

## 4. Caveats recorded honestly

1. **Repository location.** The token authenticated as `HeziLiu` cannot create repositories in that personal namespace (403 — scope does not permit repo creation). The repository was therefore created under the organization `mcpmark-eval-liuhezi`, of which the account is a member. The repository is otherwise exactly as specified: named `release-qa-evidence`, auto-initialized so it genuinely has a `main` default branch, and `runbook.md` reached `main` only through the merged PR titled exactly `Add release QA evidence run 1`, with the work developed on `evidence/run-1`.
2. **Drag-and-drop venue.** No public page was suitable (neither target page exposes draggable elements), so a trivial scratch page was stood up in the same browser session for this specific check. This is noted both here and in the warehouse row.
3. **Warehouse tenant condition.** The Snowflake tenant was expected to be brand new and empty, but the account already contained many pre-existing databases unrelated to this exercise (e.g. `RELEASE_TRACKING`, `CLUB_KITCHEN`, `AURORA_OPS`). No pre-existing object was read or modified; `release_qa` was created fresh and only it was used for this evidence pack.
4. **Console cleanliness.** The zero-message console result is reported as a genuine finding, not as "no data captured": the console was actively checked on both pages and contained nothing.
