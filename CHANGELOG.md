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
