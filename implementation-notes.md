# Designed for Self-Governance: Implementation Notes

## Opening and sharing
- `designed-for-self-governance-dashboard.html` is one self-contained file (about 190 KB). Open it in any modern browser by double-clicking. It needs no installation, account, API key or server, and it works offline.
- To share it, email the file, put it on a shared drive, or host it on any static web server (for example GitHub Pages or a lab website). A private hosted copy is also available as a claude.ai artifact.
- Document links go to the UCSF Industry Documents Library (`https://www.industrydocuments.ucsf.edu/docs/<id>`) and need internet access. All links open in a new tab.
- It uses system fonts only and has no external scripts or stylesheets. It follows the viewer's light or dark setting.

## How the horizontal timeline works
- Events sit in a scrollable row in date order. Spacing follows sequence, not scale. Year markers divide the row, and a dashed marker shows any gap of more than three years.
- Ways to navigate: drag with the mouse, swipe on touch screens, use a trackpad or shift+wheel, use the slider (kept in sync with the scroll position), use the arrow buttons (each moves 80% of the visible width), use the "Jump to" year menu, or focus the row and press the left/right arrow keys.
- Filters: the category legend chips switch each category on or off. The evidence-status chips do the same for Verified, Likely, Proposed, Industry claim and External source. A search box matches titles, descriptions, organizations, dates and document IDs.
- Hovering over or focusing a card draws arcs to related events and dims the rest. Solid arcs join two verified events; dashed arcs involve at least one unverified event.
- Clicking a card (or pressing Enter) opens a panel with:
  - detail
  - organizations
  - significance
  - quotations from that event's documents
  - document titles, dates and links
  - limitations
  - clickable related events

## Where the data lives
All content is generated from three Python files. `build.py` embeds it as JSON in the HTML.

| File | Contents |
|---|---|
| `data.py` | `EVENTS` (timeline), `QUOTES` (quotations), `THEMES` (theme names) |
| `content.py` | Overview text, theme narratives, theme-to-event mapping, limitations |
| `meta.tsv` | Archive metadata for each document ID (id, date, title, author, collection, pages, type) |
| `template.html` | Layout, styles and interactive code |

Rebuild with `python3 build.py`. This writes the dashboard, `source-documentation.md`, and `artifact.html` (a version without the document skeleton, for claude.ai publishing).

## Adding an event
Add a `dict(...)` to `EVENTS` in `data.py`:

```python
dict(id="e8805b", date="May 1988", sort="1988-05-10", cat="industry", status="verified",
  title="Short title", orgs="Who was involved",
  desc="What happened (quote only verified wording in single quotes).",
  sig="Why it matters (interpretation).", detail="Optional longer detail.",
  docs=["abcd0123"], rel=["e8802"], limits="Uncertainty, or empty string.")
```

Field rules:
- `cat` must be one of `corporate`, `industry`, `science`, `fda` or `policy`.
- `status` must be one of `verified`, `likely`, `proposed` or `claim`. An event with only `ext=[("label","https://...")]` links and no `docs` is shown as External source.
- `rel` links are made two-way automatically. The build stops with an error if `rel` names an event ID that does not exist.
- To include the event in a theme, add its ID to `THEME_EVENTS` in `content.py`.

## Adding a quotation or document
- **Quotation:** add a `dict(...)` to `QUOTES` with these fields:
  - `id`, `doc`, `page`, `theme` (`self`, `strategy`, `formation`, `fda` or `gras`), `org`
  - `cls` (`direct`, `corroborating`, `circumstantial` or `unverified`)
  - `text` (exact wording only), `sig`, `ctx`
- **Document:** add a line to `meta.tsv` for every new document ID, using the archive's own metadata. The build stops with an error if a cited ID has no metadata line.
- **Verification:** before adding a quotation, check its wording against the document. These dashboard quotations were checked by fetching each document's OCR text from the library's search index and matching after normalizing case and punctuation.

## Technical limitations
- **Static data:** the dashboard does not search the archive live. Live search would need a small server-side proxy to the library's search service (browsers block direct cross-site requests from a local file) and a way to review results before they enter the dataset.
- **Page numbers:** these are PDF pages in the archive viewer, estimated from OCR page markers. Quotation 29 also gives the transcript's printed page.
- **OCR quality:** some documents have garbled OCR. Eight quotations matched only after allowing for line numbers, hyphenation or page breaks. Check these against the page image before publishing.
- **Saved state:** the open theme is remembered in the browser where storage is allowed. Nothing else is stored, and nothing is sent anywhere.
- **Organization grouping:** the library's organization field is derived from each record's author, title and collection, so a few documents are grouped as "Other".
