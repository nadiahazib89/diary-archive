# Architecture

## Why not train a custom OCR/HTR model?

Training a handwriting recognition model (e.g. fine-tuning TrOCR) needs
hundreds of labeled image/text pairs before it produces anything usable —
data that doesn't exist yet for this diary collection, and manually creating
it would mean doing most of the transcription work twice. For a fixed,
one-time corpus (five diaries, no future additions), that cost isn't
justified.

Instead, this tool uses a general-purpose vision-capable model (Claude) for
transcription, and gets the "gets better as it goes" benefit through a
**growing context pack** rather than gradient-based learning. That's the core
design choice this document explains.

## Components

```
index.html
├── PDF handling        (pdf.js, runs entirely client-side)
├── Transcription        (fetch → Claude API, one page at a time)
├── Context pack          (names / slang / handwriting notes, JSON)
└── Persistence          (window.storage — browser-scoped key/value store)
```

### PDF handling

Scanned pages are rasterized to a canvas client-side using
[pdf.js](https://mozilla.github.io/pdf.js/), downscaled to ~900px wide before
being sent anywhere. The original PDF file never leaves the browser except as
this one compressed page image per transcription request.

### The context pack (the "learning" mechanism)

The context pack is a small JSON object with three lists:

```json
{
  "names": [{"term": "Sheila", "note": "author's wife"}],
  "slang": [{"term": "kenduri", "note": "feast/gathering"}],
  "quirks": [{"term": "...", "note": "..."}]
}
```

On every transcription request, this whole pack is rendered as plain text and
included in the prompt sent alongside the page image. After each page is
confirmed, Claude's response can propose new entries (names or Malay
terms it noticed that aren't in the pack yet); the user picks which to keep.
Accepted entries are merged into the pack and used on every page from then
on.

This means:
- Spelling of a name stays consistent for the rest of the diary once it's
  been confirmed once.
- The glossary is fully visible and editable — nothing is a black box.
- There's no training step, no GPU, no dataset to manage.

### Storage model

Everything durable is stored in the browser's own IndexedDB (via the small
[idb-keyval](https://github.com/jakearchibald/idb-keyval) helper library),
scoped to that one browser on that one device — there is no server or
database involved. Keys are kept coarse-grained on purpose — one JSON blob
per diary's pages, rather than one key per page — to stay fast and simple
even for a 400-page diary:

| Key | Contents |
|---|---|
| `diary-registry` | List of diaries added: `{id, name, totalPages}` |
| `diary-pages:{id}` | All confirmed pages for one diary: `{pageNum: {text, confirmedAt}}` |
| `progress:{id}` | Resume pointer: `{currentPage}` |
| `context-pack` | The glossary described above (shared across all diaries) |

`{id}` is derived from a hash of the file's first 64KB plus its size — not
the filename — so re-selecting the same PDF in a later session is recognized
even if it's been renamed or moved.

The raw PDF bytes are **not** persisted (they're too large for practical
browser storage across a multi-diary collection). Each session, the user
re-selects the file locally; the tool matches it against the stored registry
and resumes from the saved page.

**IndexedDB is per-browser, per-device.** It's real, durable storage — it
survives closing the tab and restarting the computer — but it doesn't sync
across browsers or devices, and it can be cleared if the user wipes site
data. The Library tab includes a **Download backup** / **Restore backup**
pair that serializes the entire registry, context pack, and every diary's
confirmed pages to a single downloadable JSON file, so users have a way to
move data between browsers or protect against data loss.

### Transcription request shape

Each request sends one page image plus a prompt asking the model to return
strict JSON:

```json
{
  "transcription": "the full transcribed text",
  "new_terms": [{"term": "...", "type": "name|slang|quirk", "note": "..."}]
}
```

Uncertain words are marked inline by the model as `[[word?]]` rather than
silently guessed, so they're easy to spot during review.

## Extending this later

Although this build assumes a fixed, known set of diaries, the same context
pack idea would extend cleanly to new material later: point the tool at a new
PDF, and it benefits immediately from whatever glossary has already been
built up, with no retraining required.
