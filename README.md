# Diary Archive

A single-page tool for transcribing scanned handwritten diaries — built for a
collection of Malaysian personal diaries from the 1970s, written mostly in
English with Malay names and slang throughout.

It runs entirely in the browser. No backend, no install, no server to pay for.

## What it does

- **Library** — add one or more scanned diary PDFs. Each is tracked
  separately, with a progress bar.
- **Transcribe** — page by page: the scan is shown next to an editable draft
  transcription. You review, correct, and confirm before moving to the next
  page ("confirm before next"). Nothing advances without your sign-off.
- **Glossary** — as you confirm pages, the tool notices recurring names and
  Malay words/slang and offers to save them. Later pages are transcribed with
  that glossary as reference, so spelling of names and terms stays consistent
  across hundreds of pages. You can also add or edit entries by hand.
- **Browse** — every confirmed page, searchable in one place.

## Setup

This tool calls Claude directly from your browser, so you need your own
Anthropic API key — separate from any claude.ai subscription, billed
pay-as-you-go.

1. Go to [platform.claude.com](https://platform.claude.com), sign in (or
   create an account), and go to **Settings → API keys** to create one.
2. Open `index.html` (or your GitHub Pages link), go to the **Settings**
   tab, paste the key in, and click **Save key**. It's stored only in your
   browser and sent only to Anthropic's API — never anywhere else.
3. Add a small amount of credit to your Anthropic account (a few dollars is
   plenty to start). Transcribing a diary page costs a fraction of a cent;
   at official September 2026 rates (~$2 per million input tokens, ~$10 per
   million output tokens), transcribing roughly 2,000 pages across five
   diaries comes out to somewhere in the low tens of dollars total — a rough
   estimate, not a guarantee, since actual cost depends on how dense each
   page is.

**A note on hosting this publicly:** if your GitHub Pages link is public,
don't treat it as a "try it live" demo to share around — the page never
stores *your* key in its source code, but anyone who opens it would need to
enter their own key to use it, and there's no reason to invite that. Keep
the link for your own use.

## Running it

Open `index.html` in a browser, or visit it via GitHub Pages. That's it —
there's no build step and nothing to install. It calls Claude's API to
produce each transcription draft and stores everything else (confirmed text,
progress, glossary) in the browser's own IndexedDB, so it works as a plain
static file with no server or database of its own.

## Using it

1. **Library tab** — drop in a scanned PDF. It's read directly in your
   browser; the file itself is never uploaded anywhere except the one page
   image sent to Claude for each transcription.
2. Click **Open** to start transcribing from wherever you last left off.
3. **Transcribe tab** — for each page, a draft transcription appears next to
   the scan. Edit the text as needed, review any suggested glossary terms,
   then **Confirm & next page**. Use **Re-run OCR** if the draft is poor, or
   **Skip page** to move on without saving anything for that page.
4. **Browse tab** — search across everything you've confirmed so far.
5. **Glossary tab** — see and edit the running list of names, Malay
   words/slang, and handwriting notes the tool is using for consistency.

5. **Library tab** — each diary has **Export .docx** and **Export .txt**
   buttons. Both pull together every confirmed page for that diary, in page
   order, with page headings, and download it straight to your computer.
   Works even if the diary's PDF isn't currently loaded — export reads only
   from confirmed text already saved, not the scan itself.

## Pausing and resuming

Your progress, confirmed transcriptions, and glossary all persist
automatically between sessions — as long as you come back in the **same
browser, on the same device**. When you return, re-select the same PDF from
the Library tab; the tool recognizes it (by file content, not just name) and
resumes exactly where you stopped, with all prior pages intact.

The raw scanned PDF itself is never stored by the tool — browser storage
isn't built to hold hundreds of megabytes of scans indefinitely. That's why
re-selecting the file each session is required.

### Backups matter here

Because everything lives in this one browser's storage, it can be lost if you
clear your browser's site data, switch browsers, or move to a new computer.
Use the **Download backup** button in the Library tab regularly — it saves a
single JSON file with all your confirmed transcriptions, progress, and
glossary. **Restore backup** loads one back in, merging it with whatever is
already there. Keep a copy of these backup files somewhere separate from the
browser itself (cloud drive, email to yourself, external drive).

## How transcription accuracy improves over time

There's no model training involved — no labeled dataset, no fine-tuning step.
Instead, every confirmed page can contribute new entries to a running
glossary (names, Malay slang, handwriting quirks). That glossary is included
as context on every subsequent transcription request, so the model has
increasingly specific information about this particular diary and author as
you progress. See `ARCHITECTURE.md` for the full design.

## Limitations

- Requires a network connection (calls the Claude API per page).
- Very large scans may take a moment to render; the tool downsamples images
  before sending them for transcription to keep things fast.
- This is a personal transcription tool, not a general-purpose OCR product —
  it's tuned for one author's handwriting and one glossary at a time.

## License

Use, modify, and adapt freely for your own archival work.
