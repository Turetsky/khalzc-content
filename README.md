# khalzc-content

Public content feed for the **Khal Zidichov Chodorov** shul app. The app reads
the bulletin and calendar PDFs from this repo over `raw.githubusercontent.com`.

This repo is the **A→B bridge** of the publishing pipeline:

```
Office Bulletin Maker ──▶ Publish tool ──PUT (GitHub API)──▶ THIS REPO ──raw──▶ App
```

Why GitHub raw (not Netlify / *.github.io): `raw.githubusercontent.com` passes the
shul's content-filter VPN on-device; `*.github.io` is unreliable per-subdomain and
Netlify production deploys cost credits. (Architecture approved Jun 2026.)

---

## Layout

```
/bulletins/<file>.(pdf|png|jpg)   weekly/periodic bulletins  — PDF *or* image
/calendars/<file>.(pdf|png|jpg)   monthly calendars          — PDF *or* image
/manifest.json                    pointer to the CURRENT bulletin + calendar
```

Both PDFs **and** images (PNG/JPG) are supported — the office often produces a
calendar as a single image. **The renderer decides how to display by the file
extension in `manifest.json`** (`.pdf` → PDF viewer; `.png`/`.jpg` → image view),
so always keep the extension on `file`.

Old files may stay in the folders for history; the app only reads whatever
`manifest.json` points at.

## manifest.json — the contract

The app fetches **only this file** first, then downloads the two files it names.

```json
{
  "bulletin": { "file": "bulletins/bulletin-2026-06-13.pdf",  "date":  "2026-06-13" },
  "calendar": { "file": "calendars/june-calendar-2026.png",   "month": "2026-06" },
  "updated":  "2026-06-13T15:04:00.000Z"
}
```

| Field            | Meaning                                                   |
|------------------|-----------------------------------------------------------|
| `bulletin.file`  | repo-relative path to the current bulletin (PDF or image) |
| `bulletin.date`  | issue date, `YYYY-MM-DD`                                  |
| `calendar.file`  | repo-relative path to the current calendar (PDF or image) |
| `calendar.month` | calendar month, `YYYY-MM`                                |
| `updated`        | ISO 8601 timestamp of the last publish                    |

> The **file extension on `file`** tells the app whether to render it as a PDF or
> an image — keep it accurate.

An empty object (`{}`) for `bulletin` or `calendar` means "none published yet" —
the app shows its last cached copy, or an empty state.

### App fetch base

```
https://raw.githubusercontent.com/Turetsky/khalzc-content/main/
```

e.g. the manifest is at
`https://raw.githubusercontent.com/Turetsky/khalzc-content/main/manifest.json`.

> Note: `raw.githubusercontent.com` caches for ~5 min (CDN). A publish appears in
> the app within a few minutes, not instantly.

## Publishing

Use the **Publish to App** tool (`web/publish.html` in the app repo). It commits
the PDF and rewrites `manifest.json` via the GitHub Contents API using a
fine-grained token scoped to this repo (Contents: Read & write). Do not commit any
token to this repo.

---

*Owner note: this repo is currently under the `Turetsky` account and is
transferable to `officekhalzc` later — update the fetch base + the publish tool's
`OWNER` constant if it moves.*
