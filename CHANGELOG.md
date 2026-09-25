# Visual changelog

Before/after screenshots of visible changes to [reportsthatmatter.org](https://reportsthatmatter.org),
kept here instead of the main [`reportsthatmatter/reportsthatmatter`](https://github.com/reportsthatmatter/reportsthatmatter)
repo so images don't bloat its history. The record for a future "how RTM got
built" write-up.

**Convention:** one entry per batch of related work, not one per screenshot —
group what belongs together the way a single PR usually does. Context first
(what was broken or missing, one line on the fix), then the images,
captioned. Include the PR/issue number so this stays traceable back to the
actual diff and reasoning. Chronological, newest at the bottom — this is a
log, not a curated highlight reel; misses (a bug in an early attempt, a
design that didn't work) belong here too.

---

## 2026-08-09 — Contents-page hierarchy, sidenote/footer overlap, sidenote length clamp

### Contents-page hierarchy ([reportsthatmatter#81](https://github.com/reportsthatmatter/reportsthatmatter/pull/81))

The contents page rendered every section — top-level parts and their
subsections alike — as one flat list. Root cause: the section splitter
already knew whether a heading was h2 or h3, it just never got threaded
through to the template. Fixed by adding a `level` field and indenting
subsections under their part.

**Before** — no way to tell "Mr. Trump's Fraudulent Elector Plan" is a
subsection of "The Results of the Investigation":

![before](2026-08-09-toc-and-sidenotes/1-contents-page-before-flat.png)

**After** — indented, de-emphasised, reads as a hierarchy:

![after](2026-08-09-toc-and-sidenotes/2-contents-page-after-hierarchy.png)

**Mobile** — same treatment holds at narrow widths:

![mobile](2026-08-09-toc-and-sidenotes/3-contents-page-after-mobile.png)

### Sidenote/footer overlap ([reportsthatmatter#81](https://github.com/reportsthatmatter/reportsthatmatter/pull/81))

Reported live on `/reports/jack-smith-vol1/mr-trumps-fraudulent-elector-plan`:
a long citation-heavy footnote overlapped the section nav and site footer.
Root cause, confirmed by measuring the DOM rather than guessing: `.sidenote`
floats right inside `.prose`, and `.prose` never cleared its floats — its
box ended ~2000px short of where the sidenote actually finished, and the
nav/footer that follow in the DOM rendered on top of it.

**Before** — nav row and footer text overlapping the still-rendering
sidenote:

![before](2026-08-09-toc-and-sidenotes/4-sidenote-footer-overlap-before.png)

**After** — `display: flow-root` on `.prose`; nav and footer correctly
render below the note:

![after](2026-08-09-toc-and-sidenotes/5-sidenote-footer-overlap-after.png)

### Sidenote length clamp ([reportsthatmatter#80](https://github.com/reportsthatmatter/reportsthatmatter/issues/80), [reportsthatmatter#82](https://github.com/reportsthatmatter/reportsthatmatter/pull/82))

The deeper problem behind the overlap above: a note much taller than its
paragraph drags every note after it out of alignment, since floats stack
top-to-bottom in the margin column independent of each note's own anchor.
Researched before building — prior art (Tufte CSS, Gwern.net's
`sidenotes.js`, native popover + anchor positioning) and real note-length
data in
[`docs/plans/2026-08-09-sidenote-design-research.md`](https://github.com/reportsthatmatter/reportsthatmatter/blob/main/docs/plans/2026-08-09-sidenote-design-research.md).
Landed on: clamp a note over 400 characters (the genuine outliers, chiefly
Jack Smith's citation blocks) to ~8 lines with a fade and a "Show full note"
toggle.

**Attempt 1 — a bug, shipped nowhere, caught by screenshotting rather than
trusting the CSS on paper.** Used `mask-image` on the whole note to fade the
truncated tail. That also faded the toggle label, since it's a child of the
masked element — label text and note text both went semi-transparent and
overlapped into an illegible mess:

![broken](2026-08-09-toc-and-sidenotes/6-sidenote-clamp-attempt1-mask-bug.png)

**Attempt 2 — fixed.** Swapped the mask for a solid gradient overlay
(`::after`, fading to the canvas colour) so the label sits on an opaque
background instead of fading with the text:

![fixed](2026-08-09-toc-and-sidenotes/7-sidenote-clamp-attempt2-fixed.png)

**Expanded** — clicking "Show full note" reveals the rest in place, using
the same checkbox the site already ships for mobile collapse:

![expanded](2026-08-09-toc-and-sidenotes/8-sidenote-clamp-expanded.png)

**Density check** — PSI's table-of-contents area, where many long notes
cluster together, holds up:

![psi](2026-08-09-toc-and-sidenotes/9-sidenote-clamp-psi-density-check.png)

**Live**, on `reportsthatmatter.org/reports/jack-smith-vol1/mr-trumps-fraudulent-elector-plan`,
post-deploy:

![live](2026-08-09-toc-and-sidenotes/10-sidenote-clamp-live-production.png)

---

## 2026-08-28 — Highlights, social proof, and search (retroactive)

Backfilling: these three features shipped between 2026-08-20 and 2026-08-22
with no screenshots logged here at the time. Captured against a local build
afterward, not at ship time — the underlying feature and output are exactly
what's live in production, only the capture is after the fact.

### Highlight-to-share and quote anchors ([reportsthatmatter#94](https://github.com/reportsthatmatter/reportsthatmatter/issues/94), [reportsthatmatter#95](https://github.com/reportsthatmatter/reportsthatmatter/issues/95))

Selecting part of a paragraph opens a popover to copy a link, copy the quote,
or save the highlight. The link names the words themselves, not just the
paragraph:

![selecting a phrase opens the share popover](2026-08-28-highlights-social-proof-search/1-highlight-select-share-popover.png)

Following that link marks exactly the quoted words, nothing more:

![following the link marks exactly those words](2026-08-28-highlights-social-proof-search/2-quote-link-marks-exact-words.png)

Saved highlights are kept in the browser (never sent anywhere) and listed at
`/highlights`, with the quote, source, and an export:

![the /highlights page](2026-08-28-highlights-social-proof-search/3-highlights-page.png)

### Social proof ([reportsthatmatter#96](https://github.com/reportsthatmatter/reportsthatmatter/issues/96))

What other readers marked, shown the same way any highlight is shown — a
wash, not an underline or a printed count — with intensity scaled by how
many readers marked it. Six readers:

![strong wash, 6 readers](2026-08-28-highlights-social-proof-search/4-social-proof-strong-wash-6-readers.png)

One reader, faint by comparison:

![faint wash, 1 reader](2026-08-28-highlights-social-proof-search/5-social-proof-faint-wash-1-reader.png)

The same data surfaces on the report's contents page, ranked:

![Most marked passages on the contents page](2026-08-28-highlights-social-proof-search/6-most-marked-passages.png)

### Full-text search ([reportsthatmatter#100](https://github.com/reportsthatmatter/reportsthatmatter/issues/100))

Search spans every report, not just the one you're reading. A result is the
matched passage itself, with the term highlighted and the report, section,
and printed page it's on:

![search results, term highlighted, across reports](2026-08-28-highlights-social-proof-search/7-search-results.png)

---

## 2026-08-28 — Leveson page-flow repair ([reportsthatmatter@f07a860](https://github.com/reportsthatmatter/reportsthatmatter/commit/f07a860))

On the *Operation Glade* page, a running header was rendered as prose and
ordinary continuation lines were mistaken for an indented quotation. The
ingest now respects the original volume boundaries, removes repeated page
furniture, and retains only genuine quotations.

**Before** — the header interrupts paragraph 2.1, while the continuation is
set as a quotation:

![before](2026-08-28-leveson-ingest-layout/before-operation-glade.png)

**After** — the paragraph reads continuously; page 254 remains a marker and
the following numbered paragraph also flows correctly:

![after](2026-08-28-leveson-ingest-layout/after-operation-glade.png)

---

## 2026-08-29 — The Columbia report, and a two-column layout read three ways wrong

### Columbia Accident Investigation Board report ([reportsthatmatter#37](https://github.com/reportsthatmatter/reportsthatmatter/issues/37), [#101](https://github.com/reportsthatmatter/reportsthatmatter/issues/101))

The sixth report, and the first set in two columns. It had been ruled out a
year earlier for exactly that reason: `pdftotext -layout` puts both columns on
the same physical line, so reading line by line welds an unrelated sentence
into the middle of every paragraph. The fidelity checks never saw it — they
count words rather than order them.

Getting it right took four passes, and the first three shipped. Worth
recording, because each failure looked like success:

1. **Detection thresholds too strict.** A minimum gutter width of four
   characters and a blankness threshold near 1.0 missed 86 of 248 pages,
   including the executive summary. Columbia went live reading *"the February
   1, 2003, loss of the Space management across program elements"*.
2. **The column boundary was not a hard break.** Once split, the foot of the
   left column and the head of the right sat adjacent in the block stream, and
   the paragraph-continuation rule joined them back together.
3. **Justified text spills into the gutter.** A long word at the end of the
   left column reaches a character or two into the band, so those lines counted
   as full-width and were never split — welding the columns back on exactly
   the lines where the left column runs longest. The published summary read
   *"In the process, Columbiaʼs control over specifications and requirements,
   and waivers tragedy was compounded"*: three clauses from two columns in one
   sentence.

Each line is now split at its own run of whitespace nearest the gutter, and
the boundary between columns stops anything being joined across it.

**After** — the executive summary, reading in column order:

![Columbia executive summary reading correctly](2026-08-29-columbia-two-column/reading-view.png)

**The contents page**, 106 sections deep:

![Columbia contents page](2026-08-29-columbia-two-column/contents.png)

**The archive**, now six reports:

![Archive page with six reports](2026-08-29-columbia-two-column/archive.png)

The lesson is the one already in `AGENTS.md` and worth restating: every
fidelity gate passed on all three broken versions. Only opening the published
page caught them.

---

## 2026-09-01 — Half of Litvinenko's paragraphs were not quotations

### Hanging-indent paragraphs read as block quotes ([uk-litvinenko-inquiry#1](https://github.com/reportsthatmatter/uk-litvinenko-inquiry/issues/1))

Reported as one paragraph — 3.76 shown as a quotation when it is ordinary
prose. It was **865 of the report's 1,089 numbered paragraphs**, each cut in
half: the first line kept as prose, the remainder turned into a block quote.
Half of every blockquote in the document was an artefact.

The report sets a numbered paragraph with a hanging indent — the number at the
left edge, the text inset by six — and the pipeline measured the document
margin from *raw* page lines, where footnote blocks also sit at the edge. The
margin came out at 0 instead of 6, and anything indented five past the margin
reads as a quotation. The multi-volume path already measured the cleaned page
body; only the single-volume path did not, which is why Leveson never showed
it.

Fixing that then took the report's **real** quotations away: it sets body text
at 7 and quotations at 10, and the rule demanded five, so the page went from
half its paragraphs wrongly quoted to no quotations at all. Lowering the
threshold globally was worse again — at three, Challenger turned 442
paragraphs into quotations and Columbia 203 — so the inset is now a property
each report declares, like its geometry.

**After** — 3.76 reads as prose, and the one genuine quotation on the page is
still a quotation:

![Chapter 2 with paragraphs whole and one real quotation](2026-09-01-litvinenko-blockquotes/after.png)

Litvinenko: 1,735 → 559 blockquotes, retention 99.2% → 99.3%, words not found
in the source 22 → 0. Every other report byte-identical.


---

## 2026-09-03 — Four more reports: 9/11 Commission, Deepwater Horizon, Philip Morris, Hillsborough

The archive went from six reports to ten
([reportsthatmatter#124](https://github.com/reportsthatmatter/reportsthatmatter/pull/124)).
All four are born-digital PDFs with a clean text layer, picked off the report
backlog and each given its own repo under the org. Retention 96.6–99.7%.
Screenshots here are the shipped state on reportsthatmatter.org, not a
before/after — there was nothing before.

### The 9/11 Commission Report ([reportsthatmatter#85](https://github.com/reportsthatmatter/reportsthatmatter/issues/85))

585 pages. Every page of the PDF opened with an Adobe InDesign output slug —
`Final1-4.4pp 7/17/04 9:12 AM Page 13` — that running-furniture detection
can't catch (the date, time and page token change every page) and that is the
*only* place the printed page number appears. An inline `productionSlug` pass
in the report's own `ingest.ts` reads the number off it and drops the line.

![9/11 Commission — contents page](2026-09-03-four-more-reports/us-911-commission-contents.png)

![9/11 Commission — inside chapter 1, with a printed-page marker](2026-09-03-four-more-reports/us-911-commission-reading.png)

### Deep Water — the BP Deepwater Horizon commission ([reportsthatmatter#87](https://github.com/reportsthatmatter/reportsthatmatter/issues/87))

386 pages, 775 footnotes lifted into the margin as sidenotes, with the
over-long ones clamped and a "show full note" toggle.

![Deep Water — contents page](2026-09-03-four-more-reports/us-deepwater-horizon-contents.png)

![Deep Water — a sidenote in the margin](2026-09-03-four-more-reports/us-deepwater-horizon-reading.png)

### United States v. Philip Morris ([reportsthatmatter#33](https://github.com/reportsthatmatter/reportsthatmatter/issues/33))

Judge Kessler's 1,682-page RICO opinion — the largest single document in the
archive. The ECF header stamp (`Case 1:99-cv-02496-GK  Document 5750  Filed
09/08/2006  Page 100 of 1682`) strips as running furniture once its digits are
blanked. The numbered findings of fact each get a stable, text-derived
permalink.

![US v. Philip Morris — contents page, 129 sections](2026-09-03-four-more-reports/us-v-philip-morris-contents.png)

![US v. Philip Morris — a finding of fact with its citation sidenote](2026-09-03-four-more-reports/us-v-philip-morris-reading.png)

### The Report of the Hillsborough Independent Panel ([reportsthatmatter#90](https://github.com/reportsthatmatter/reportsthatmatter/issues/90))

389 pages, 99.7% retention — the cleanest ingest in the archive by the word
count, and the main body (all 12 chapters, decimal-numbered paragraphs) reads
well. `quoteInset(10)` was needed to stop the front-matter summary's
hanging-indent numbered list being severed into blockquotes — the same failure
family as the Litvinenko defect above, 65 paragraphs affected.

**The miss:** this report sets its section headings as colour and weight with
no textual marker, so `pdftotext` flattens them into ordinary lines. The
structure pass finds almost nothing real — 9 "sections", most of them spurious
ALL-CAPS quoted document titles ("OF ACTION: CHECK TRANSCRIPTS…", "18. 'TO HER
MAJESTY'S ATTORNEY GENERAL…"). The contents page below is the result. `/full`
is completely fine; section navigation is not. Tracked as
[reportsthatmatter#125](https://github.com/reportsthatmatter/reportsthatmatter/issues/125);
shipped on the strength of `/full`.

![Hillsborough — contents page, sectioned badly by undetectable headings](2026-09-03-four-more-reports/uk-hillsborough-panel-contents.png)

![Hillsborough — the whole-report view, which reads correctly](2026-09-03-four-more-reports/uk-hillsborough-panel-reading.png)

---

## 2026-09-11 — Every page gets a real share-card image

Almost the whole site had no og:image — a link shared to Slack, iMessage, or X previewed as bare text everywhere except five curated quotes in one report (jack-smith-vol1). Added a second card layout — a title and subtitle, no quote — and generated one for the site itself and one per report, so a shared link always has something to show: a curated quote where one exists, that report's own card otherwise, the site's card for anything with no report at all (the homepage, `/about`, `/search`, a report's own contents page). Beads: `reportsthatmatter-obw`; commit `aa08be1`.

**The site's own card** — the homepage's headline and standfirst, reused verbatim so the card and the page it fronts can't say different things:

![Reports that Matter — the site's default share card](2026-09-11-social-preview-cards/1-site-card.png)

**A report's default card** — what a report's contents page, its `/full` page, and any paragraph link that isn't one of the curated quotes now show instead of nothing:

![Investigation of the Challenger Accident — its default share card](2026-09-11-social-preview-cards/2-report-card-challenger.png)

![Wall Street and the Financial Crisis: Anatomy of a Financial Collapse — its default share card](2026-09-11-social-preview-cards/3-report-card-financial-crisis.png)

---

## 2026-09-13 — Report plates and the pilcrow favicon

Issue [reportsthatmatter#99](https://github.com/reportsthatmatter/reportsthatmatter/issues/99). Every report now has a plate: one treated image drawn from the report's own evidence (the exhibit, not the event), in its archive row, above its title, and on its share cards. The navbar drops the seal, whose lettering was 3.6px a character at 30px, for the bare wordmark; the favicon becomes a pilcrow in the seal. Study, sourcing, and the three places the live version departs from the mockups: `docs/design/2026-09-12-imagery/README.md` in the main repo.

**Before** — the archive, text only, with the seal beside the wordmark:

![before](2026-09-13-report-plates/1-archive-before.png)

**After** — each report with its plate in a fixed 92px slot; the header is the wordmark alone:

![after](2026-09-13-report-plates/2-archive-after.png)

**Phone** — the plate shrinks to a 64px slot beside the title; with the seal gone the full wordmark shows, and the nav wraps under it. Before and after:

![before](2026-09-13-report-plates/3-archive-phone-before.png)
![after](2026-09-13-report-plates/4-archive-phone-after.png)

**Report page** — Columbia's contents page, before and after: the impact hole in RCC Panel 8 from the final foam test, set above the title as a frontispiece:

![before](2026-09-13-report-plates/5-contents-before.png)
![after](2026-09-13-report-plates/6-contents-after.png)

**Share card** — Columbia's default card, before and after:

![before](2026-09-13-report-plates/7-card-before.png)
![after](2026-09-13-report-plates/8-card-after.png)

**A miss along the way** — the site's own card briefly carried the pilcrow-in-seal in the plate's slot, and its standfirst ran 41px off the bottom edge, clipping the footer. Shipped without it, and `scripts/cards.mjs` now fails any card that overflows.

**Favicon** — the pilcrow as first drawn sat 9.6 units below centre; measured and re-set so its ink box centres in the seal:

![favicon](2026-09-13-report-plates/9-pilcrow-512.png)

---

## 2026-09-14 — Navbar icon, a press page, and two weak plates replaced

Follow-up to [reportsthatmatter#99](https://github.com/reportsthatmatter/reportsthatmatter/issues/99). The pilcrow-in-seal mark returns to the navbar beside the wordmark — a different, simpler asset than the lettered ring-seal dropped from there on 2026-09-13, and already proven legible to 16px as the favicon. New `/press` page with the logo, the logotype, and the two together, at the sizes the site actually ships. And two plates replaced: Columbia's read as an unidentifiable dark blob (flagged directly — "looks like it should be Hillsborough"), and Challenger's report has no embedded photographs anywhere in its own PDF, so its plate and "cut" alternate were both the same hand-drawn diagram.

**Navbar** — wordmark alone, then with the mark beside it:

![before](2026-09-14-navbar-press-plates/1-nav-before.png)
![after](2026-09-14-navbar-press-plates/2-nav-after.png)

**Press page** — new, `/press`:

![press page](2026-09-14-navbar-press-plates/3-press-page.png)

**Columbia** — the previous plate (p.82, the RCC Panel 8 impact hole) at archive-row size, next to the report page header, before and after switching to p.98 (the shuttle in the Vehicle Assembly Building door, the same subject the previous entry's cut candidate used, rendered as a plate to stay consistent with the other nine):

![before](2026-09-14-navbar-press-plates/4-columbia-row-before.png)
![after](2026-09-14-navbar-press-plates/5-columbia-row-after.png)
![before](2026-09-14-navbar-press-plates/6-columbia-header-before.png)
![after](2026-09-14-navbar-press-plates/7-columbia-header-after.png)

**Challenger** — the O-ring joint diagram, archive row and header, replaced with NASA's own photograph of the smoke plume at the right booster's aft field joint at liftoff (hosted on nasa.gov, public domain) — the photographic evidence the Rogers Commission used to locate the failure, sourced externally the same way Leveson and Philip Morris were:

![before](2026-09-14-navbar-press-plates/8-challenger-row-before.png)
![after](2026-09-14-navbar-press-plates/9-challenger-row-after.png)
![before](2026-09-14-navbar-press-plates/10-challenger-header-before.png)
![after](2026-09-14-navbar-press-plates/11-challenger-header-after.png)

---

## 2026-09-18 — Challenger appendix OCR review

Bead `reportsthatmatter-agk`; [report commit `847850a`](https://github.com/reportsthatmatter/challenger-accident/commit/847850a9a85b7772cd50e51903fa0f0f2a9e5ef9). The first prose batch from Challenger's OCR suspect queue was checked against the GPO scan. Eighteen suspect tokens leave the queue through 25 page-scoped corrections: damaged footnote numbers and definitions, an O-ring task-force title, and a table header now agree with the printed report.

**Before** — on printed page 211, OCR rendered footnote 25 as `2s` both in Mulloy's quoted statement and in the citation line:

![before](2026-09-18-challenger-ocr/1-before-page-211.png)

**After** — both references read `25`; the same review-and-scan method covers the rest of this batch:

![after](2026-09-18-challenger-ocr/2-after-page-211.png)

---

## 2026-09-18 — The Bloody Sunday Inquiry

New report ([reportsthatmatter#143](https://github.com/reportsthatmatter/reportsthatmatter/pull/143), `@rtm/ingest` v0.12.16). Volume I of Lord Saville's report sets its notes beneath each paragraph, numbered from 1 again every time and in two columns. It quotes 1972 telegrams in capitals, and its facing pages sit at different margins. Read the usual way, 392 references pointed at one note and every right-hand page was cut into single-line paragraphs. Four report-declared passes read it as printed. No "before" images: the report was never published in its broken state.

**Contents page** — the plate is a Humber APC photographed on Bloody Sunday, from the report's own Glossary:

![contents](2026-09-18-saville/1-contents-page.png)

**Paragraph notes** — 9.165's two notes sit beside it with their printed numbers, under David's telegram, which stays a quotation:

![telegram and notes](2026-09-18-saville/2-telegram-and-paragraph-notes.png)

**Chapter contents** — Chapter 8's title comes out whole ("…August to December 1971"), where the source wraps it, and its own contents list its subsections by paragraph:

![chapter contents](2026-09-18-saville/3-chapter-contents.png)

**Mobile:**

![mobile](2026-09-18-saville/4-telegram-mobile.png)

**Archive row:**

![row](2026-09-18-saville/5-archive-row.png)

---

## 2026-09-25 — A report's landing page: background, findings, where to start

### Landing pages for Jack Smith and Wall Street and the Financial Crisis ([reportsthatmatter#154](https://github.com/reportsthatmatter/reportsthatmatter/pull/154))

A report's first page was only its table of contents. A reader arriving cold had no idea what the report was about, when it was written, or why it mattered. Readers who were sent links said as much. The two reports with an introduction of ours now open on a landing page: the year in the header, a standfirst, the background, what the report found (with a few key passages quoted), and where to start reading, all above the contents. Our words are set in the site's sans and the report's in its serif, so it is visible whose words are whose. Every quotation is checked word for word against the report at build time. Reports without an introduction keep the plain contents page.

**Before** — the report opened on its contents:

![Jack Smith report, before](1-before-jack-smith.png)

**After** — header with year and standfirst, then background:

![Jack Smith report, after: header](2-after-jack-smith-header.png)

**What it found**, with key passages linked to the exact words:

![What it found](3-after-what-it-found.png)

**Where to start reading**, section by section:

![Where to start reading](4-after-where-to-start.png)

![Wall Street and the Financial Crisis, after](5-after-wall-street-header.png)

Rufus's highlights are now in the marks table, so the passages show as marked in the text. (A landing page itself carries no highlights section; see reportsthatmatter#155.)

![A seeded highlight in the text](6-after-highlight-in-text.png)
