# Research Report — Handmade Font Family (Phase 0)

Date: 2026-10-29 · Platform: Windows · Tool budget: ≤ $100 · Target: 2026-12-18

Covers the eight questions in the Phase 0 brief. Every recommendation is a decision I made,
with the alternatives I rejected. Sources at the bottom.

---

## Q1. Toolchain on Windows

### Candidates evaluated

| Tool | Native source format | UFO | `.fea` editing | Build path | Beginner curve | Cost |
|---|---|---|---|---|---|---|
| **Fontra Pak** | `.fontra` / **`.ufo` / `.designspace` (native)** | native R/W | yes (feature editing in-app) | fontmake | moderate, modern UI | **$0** |
| **FontCreator 16** | `.fcp` (proprietary binary) | import/export only | yes — OpenType Designer + AFDKO `.fea` code editor | internal exporter | **easiest** | Essentials $49 / Pro $199 |
| FontLab 8 | `.vfc` / `.vfj` | import/export | yes | internal | steep | ~$459 |
| FontForge | `.sfd` | import/export | yes (low-level) | internal | brutal | $0 |
| Glyphr Studio | own/browser | no | limited | limited | easy | $0 |

### Decision: Fontra Pak + fontmake + HarfBuzz, $0

**Editor:** Fontra Pak. **Source of truth:** UFO + designspace on disk.
**Features:** hand/agent-authored `.fea` text files, assembled into the UFO.
**Build:** `fontmake` (pip, Python ≥ 3.10, works on Windows) → OTF/TTF, then WOFF2.
**Proof:** HarfBuzz `hb-shape` / `hb-view` for machine-checkable shaping, plus a local
browser proof page for visual review.

**Why.** This project is unusual: the font is largely *generated*. Hundreds of `calt` rules
will be emitted from the combination tracker, not clicked into a GUI. That makes `.fea` and
UFO first-class text artifacts that must live in git and be writable by scripts. Fontra is
the only candidate in budget whose **native** format is UFO — no round-trip, clean diffs,
agents can read and write sources directly. It also has background-image support (PNG/JPEG,
UFO-compatible), which is exactly what tracing scanned letters needs.

**Rejected, and why:**

- **FontCreator 16 Essentials ($49)** — genuinely the friendliest Windows editor, and the
  author of the best connecting-script tutorial in existence. Rejected as *primary* because its
  native project file is a proprietary binary (`.fcp`); UFO is interchange only, so
  round-tripping agent-generated sources risks silent data loss and produces useless git diffs.
  Also **interactive proofing is Professional-only ($199)** — over budget — and proofing is the
  single most valuable feature for a connecting font.
  → Kept as the **documented escape hatch** if the Fontra spike fails. Read its tutorial regardless.
- **FontLab 8 (~$459)** — over budget, and overkill for a beginner.
- **FontForge** — free, but a brutal learning curve for someone new to type design, and `.sfd`
  is less standard than UFO.
- **Glyphr Studio** — pleasant for beginners, but no UFO and not enough feature-code control.

**Pricing caveat:** sources conflict on FontCreator 16 — the official comparison chart shows
Essentials $49 / Professional $199; a third-party review states $149 flat. Verify on the vendor
store before any purchase. Nothing is being bought right now.

**Phase 1 spike required:** confirm Fontra Pak → UFO → fontmake → hb-shape round-trips on three
letters before committing drawing time. See spike `S1` in the roadmap.

---

## Q2. Joining model for the cursive font

### How connecting Latin is actually engineered

Three known approaches:

1. **"Meet halfway" / single join height.** Every letter starts and ends with a connecting stub
   at one fixed height, usually about halfway between baseline and x-height. Because all entries
   and exits share a height, *every* pair connects with no feature code at all.
2. **Positional alternates via `calt`.** Each letter gets `.start` / `.between` / `.end`
   variants; contextual-alternate lookups swap them by word position. This is the FontCreator
   tutorial method.
3. **Separate stroke glyphs.** Letters reduced to invariant cores, with entry/exit/connector
   strokes as their own glyphs. Most flexible, far too much machinery for this timeline.

The "meet halfway" model has one well-documented failure mode: **b, o, v and w naturally exit
high** (at x-height, not mid-height). Forcing their exits down to a single low join makes them
dip unnaturally far toward the baseline — the exact tell that makes cheap cursive fonts look
wrong. The letter **e** also distorts, because it must curve away from whatever precedes it and
ends up slanting less than its neighbours.

### Decision: geometric defaults at **two** join heights, plus `calt` only for the high-exit case

- **Join low** ≈ 40–50% of x-height. The default for the 22 letters that exit low.
- **Join high** ≈ x-height. Used by the exits of **b, o, v, w**.
- Every lowercase letter carries a built-in **entry stub** and **exit stub** terminating at an
  exact, specified coordinate. Sidebearings are fixed, not per-glyph.
- Consequence: **all 676 bigrams connect by geometry alone, with zero feature code.** The brief's
  requirement — every combination renders with a working default connection, generated by rules
  not per-pair drawing — is satisfied by the drawing constraint itself, which is cheaper and more
  robust than satisfying it with lookups.
- The only systematic feature work: letters following `b o v w` need an alternate with a **high
  entry stub**. One `calt` rule over a glyph class handles all of them:
  `@HIGH_EXIT = [b o v w]` → substitute the next letter with its `.hi` form.

**Why two heights, not one:** one height produces the documented b/o/v/w failure. **Why not
three or more:** each additional height multiplies the alternate sets and the review burden; two
is the most a 7-week budget absorbs. Revisit only if round-1 proofs demand it.

**`curs` (GPOS cursive attachment) — rejected.** HarfBuzz implements GPOS lookup type 3 properly,
so browsers would be fine. But support is unreliable in DirectWrite/Word and Adobe for *Latin*
text, and more fundamentally `curs` only *moves* glyphs — it cannot reshape a stroke. It can't
fix a letter whose exit is drawn at the wrong angle. Geometry plus `calt` does strictly more,
everywhere.

**`calt` vs `rclt` — use `calt`.** It is on by default in browsers, Adobe apps and Figma. `rclt`
is nominally always-on but has patchier real-world support. Neither is on by default in Word —
which is survivable precisely because our defaults are geometric (see Q7).

### Paper constraint that makes this work

The template sheet prints **dashed join-height lines** and **tick marks on the left and right
edges of every glyph box**. A letter is only valid if its entry stroke crosses the left edge
exactly at its entry tick and its exit stroke crosses the right edge exactly at its exit tick.
That single rule is what makes hand-drawn letters align digitally. Full spec in the architecture
doc.

---

## Q3. Exception mechanisms

**Contextual alternate (`calt`) — the default mechanism.** Swaps one letter for a variant based
on its neighbours. Preserves one-glyph-per-letter editing, keeps spacing and cursor behaviour
normal, and is individually revertible.

**Ligature (`liga` / `dlig`) — only when two letters must genuinely merge into one drawn shape**
(a shared stroke or crossbar, e.g. `tt`, `ff`, or an `oo` that wants one continuous loop). Real
costs: breaks text selection and cursor granularity inside the pair, removes any kerning between
the two, and the user cannot selectively undo it. Use sparingly and deliberately.

**Rule of thumb:** if the fix is "this letter should look different here," use `calt`. If the fix
is "these two letters are one drawing," use a ligature.

**Word-initial and word-final forms — deferred, on purpose.** Because every letter carries entry
and exit stubs, a word already begins with a natural lead-in stroke and ends with a natural exit.
These read as correct cursive. Dedicated `.init` / `.fina` forms are a refinement, not a
requirement, so **the MVP ships with no positional forms at all.** This is the single largest
scope saving available and it costs almost nothing visually.
(For reference, when they are added later: make the initial form the default glyph and substitute
to the connecting form when preceded by a letter — `.fea` has no negative lookbehind, so the
default-is-initial direction is the expressible one.)

**Context scope decision:** **bigrams plus word boundaries.** Trigrams and longer contexts only
when a specific case is flagged during proof review and recorded in the tracker. Exhaustive
trigram coverage would be 17,576 cells and is explicitly out of scope.

---

## Q4. Combination tracker

Design summary; full schema in the architecture doc. Nothing is built this phase.

- **One source of truth:** `sources/data/bigrams.csv`, plus `boundaries.csv`. CSV because it
  diffs cleanly in git, is readable without tooling, and is trivially machine-parseable.
- **Explicit combinatorics:** 26 × 26 = **676 ordered bigrams**, plus 26 word-initial and 26
  word-final positions = **728 review cells**.
- **In scope:** every one of the 676 gets a visual review at least once.
- **Out of scope:** trigrams (17,576), and adjacency involving capitals, digits or punctuation in
  the cursive font.
- **Critical framing (per the brief):** the tracker records *potential exceptions*. It is not a
  to-do list. Defaults already cover all 676. A row's resting state is "default is fine."
- Feeds three Phase-2+ tools: a `.fea` generator, a proof-sheet renderer for all 676 combinations,
  and a coverage report.

---

## Q5. Character set

### Cursive font (Font 1)

**MVP:** `a`–`z` (26) · the `.hi` entry alternates for letters following `b o v w` · `space` ·
non-connecting basic punctuation `. , ' - ! ?`

**Capitals and digits — borrow, don't omit.** A missing glyph renders as `.notdef` (a box), which
looks broken. Decision: the cursive font borrows **non-connecting capitals and digits from Font
2**. One drawing effort serves both fonts. They must be drawn to harmonise in weight and slant
with the cursive — a stated constraint on Font 2, not an afterthought.

**Accents — worth it, and cheap.** Accents sit above the letter and never touch the join, so they
carry near-zero risk to the connection system, and are built as components over existing base
letters.

- **MVP French set:** `é è ê à ç` — the five highest-frequency.
- **Stretch:** `ë î ï ô ù û â œ` plus guillemets `« »`.

### Simple font (Font 2)

**MVP:** `A`–`Z`, `a`–`z`, `0`–`9`, basic punctuation.
**Stretch:** the French accented set, matching Font 1's.

---

## Q6. Paper-to-digital process

Full spec in the architecture doc. Headlines:

- **Template:** printed glyph boxes at a known physical size (so scan scale is deterministic),
  with baseline / x-height / cap-height / ascender / descender guides, **two dashed join-height
  lines**, edge tick marks at the join heights, a slant guide, and a per-box label `S01-a`.
- **Scan:** 600 dpi, **grayscale** (not bitonal), flatbed, auto-contrast and auto-crop **off**.
  PNG out.
- **Trace: manually, in Fontra, over the scan as a background image.** Auto-trace is rejected as
  the primary path — Inkscape Trace Bitmap on inked pen texture yields outlines with hundreds of
  nodes and ragged edges, which is fatal for a font whose joins must land on an exact coordinate.
  Auto-trace stays available as a reference underlay.
- **Cleanup standard:** low node count, points on extrema, no overlapping contours within a glyph,
  consistent terminal shapes, and the hard rule — **exit stub terminates exactly at (advance
  width, join height); entry stub starts exactly at (0, join height).**
- **Documentation:** a lean dated `docs/journal.md` plus the decision log. Scans are archived as
  the primary process evidence; no heavy write-up.

---

## Q7. Target environments and what "works" means

| Environment | `calt` default | Default joins work? | Exceptions fire? |
|---|---|---|---|
| Chrome / Firefox / Safari (web, WOFF2) | **on** | yes | yes |
| Figma | **on** | yes | yes |
| Adobe InDesign / Illustrator / Photoshop | **on** | yes | yes |
| Windows apps via DirectWrite (Notepad, WordPad, etc.) | on | yes | yes |
| **Microsoft Word** | **OFF** — `enableOpenTypeFeatures` is off by default; user must tick Font ▸ Advanced ▸ *Use Contextual Alternates* | **yes** | only after the user ticks the box |
| **Google Docs** | n/a | **cannot install custom fonts at all** | no |

**This is the payoff of the geometric model.** Because connections live in the outlines rather
than in lookups, the font still connects every letter in Word with contextual alternates switched
off. Only the refinements are lost. A `calt`-dependent design would simply fall apart there.

Google Docs is **out of scope** — it accepts no custom font uploads outside enterprise Workspace
font administration. Workaround to document, not to engineer: compose elsewhere and place as PDF
or image.

### Definition of "works" — the test tiers

- **Tier A (must pass to ship):** all 676 bigrams connect visually in Chrome, **and** in Word with
  `calt` off.
- **Tier B (should pass):** every shipped exception fires correctly in Chrome, Figma, Adobe, and
  Word with `calt` on. Verified by `hb-shape` assertions, not by eye alone.
- **Tier C (nice):** accented glyphs render correctly; WOFF2 loads and renders in all three major
  browsers.

---

## Q8. Spacing, kerning, and the risk register

**Spacing.** The exit stub *is* the letter spacing. Therefore all connecting lowercase glyphs take
**fixed, uniform sidebearings** (left sidebearing 0, with the stub carrying the gap). Any
per-glyph sidebearing tuning on a connecting letter tears the join. Spacing is adjusted by
redrawing stub length, never by nudging sidebearings.

**Kerning.** **Do not kern connecting pairs — at all.** Kerning translates glyphs horizontally,
which pulls joins apart. Kerning is permitted only between non-connecting glyphs: capitals,
digits, punctuation, and letter-to-punctuation pairs.

### Top risks

| # | Risk | Severity | Mitigation |
|---|---|---|---|
| 1 | Hand-drawn joins land off the exact coordinate → visible kinks across the whole font | **High** | Edge tick marks on the template; a Phase-1 **validator script** asserting every glyph's entry/exit node position in the UFO |
| 2 | **Two fonts in ~7 weeks** is aggressive | **High** | Cut order published up front (below). Font 2 is the schedule buffer, not Font 1 |
| 3 | `b o v w` high exits — the #1 documented cursive-font tell | Med-High | Two join heights + `.hi` alternates, decided above |
| 4 | Fontra is young; beginner may stall on its UI | Medium | Timeboxed Phase-1 spike `S1`; FontCreator Essentials ($49) is the pre-approved escape hatch |
| 5 | `e` distorts against preceding letters | Medium | Explicit review item in drawing round 1; candidate for an early alternate form |
| 6 | Traced outlines bloat to hundreds of nodes → inconsistent joins | Medium | Manual tracing standard + node-count ceiling in the cleanup spec |
| 7 | Overlapping strokes at joins render as blobs | Medium | Keep stubs non-overlapping by construction; check in proof sheets |
| 8 | The 676-cell tracker is mistaken for a to-do list → false sense of scope | Low-Med | Status lifecycle defaults to "default is fine"; coverage report reports *reviewed*, not *completed* |

### Scope cut order, if the date is threatened

1. Font 1 stretch accents → 2. Font 2 accents → 3. all drawn exceptions (defaults still work) →
4. Font 2 digits/punctuation → 5. Font 2 entirely.

**Never cut:** the 26 lowercase letters, the two join heights, the `.hi` alternate set, the Tier A
proof.

---

## Sources

- [Fontra Pak releases (2026.10.0)](https://github.com/fontra/fontra-pak/releases/tag/2026.10.0) · [Fontra docs — intro](https://docs.fontra.xyz/introduction/) · [opening UFO/designspace](https://docs.fontra.xyz/how-tos/opening-ufo-designspace/) · [background image](https://docs.fontra.xyz/reference/editor-view/canvas/glyph-editor/background-image)
- [FontCreator comparison chart](https://www.high-logic.com/font-editor/fontcreator/comparison-chart) · [features and benefits](https://www.high-logic.com/font-editor/fontcreator/features-and-benefits) · [FontCreator 16.0 release notes](http://www.softerviews.org/FontCreator.html)
- [High-Logic — Creating Connecting Handwritten Fonts](https://www.high-logic.com/font-editor/fontcreator/tutorials/creating-handwritten-fonts-with-contextual-alternates)
- [Cursive Workshop — Why so many cursive fonts disappoint](https://cursiveworkshop.com/article/why-so-many-cursive-fonts-disappoint-and-what-distinguishes-good-fonts)
- [Glyphs — Features, part 2: contextual substitutions](https://glyphsapp.com/learn/features-part-2-contextual-substitutions) · [Tuts+ — How to Design and Construct a Functional Cursive Font](https://design.tutsplus.com/tutorials/how-to-design-and-construct-a-functional-cursive-font--cms-23295)
- [OpenType spec — GPOS (cursive attachment, lookup type 3)](https://learn.microsoft.com/en-us/typography/opentype/spec/gpos) · [FontLab blog — GSUB, GPOS and HarfBuzz](https://blog.fontlab.com/2026/03/03/gsub-gpos-and-harfbuzz/)
- [HarfBuzz utilities (hb-view / hb-shape)](https://harfbuzz.github.io/utilities.html) · [FontGoggles](https://glyphsapp.com/tools/fontgoggles) (macOS only — hence the hb-* + browser proof stack)
- [fontmake](https://github.com/googlefonts/fontmake) · [ufo2ft](https://pypi.org/project/ufo2ft/)
- [vLetter — Enable ligatures in Microsoft Word](https://www.vletter.com/help/font-faq/enable-ligatures-in-microsoft-word.html) · [Laura Worthington — Microsoft Word support](https://lauraworthingtondesign.com/support/how-do-i/microsoft-word) · [MDN — font-variant-ligatures](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/font-variant-ligatures)
- [Inkscape Trace Bitmap settings](https://perfectvector.com/blog/inkscape-trace-bitmap) · [Capturing Handwriting (Evil Mad Scientist)](https://wiki.evilmadscientist.com/Capturing_Handwriting)
- [Comparison of font editors (Wikipedia)](https://en.wikipedia.org/wiki/Comparison_of_font_editors)
