# Phased Roadmap & Epics — fontspace

Target: **2026-12-18** (~7 weeks from 2026-10-29). Level 3, 8 epics.

**Legend** — 🎨 owner at the drawing table · 🤖 agent-only · 🤝 both · ⚠️ needs owner approval

Week numbers are indicative, not contractual. The critical path is **Epic 2 → 3 → 4**; everything
else can slip into the buffer.

---

## Phase 1 — Tooling setup 🤖
**Epic 1 · Prove the pipeline before any drawing time is spent.** Week 1.

| ID | Story | Who |
|---|---|---|
| 1.1 | **Spike S1:** install Fontra Pak, create a UFO, draw three crude letters (`a`, `b`, `n`), build with fontmake, shape with `hb-shape`. Does a 3-letter word connect? | 🤖 |
| 1.2 | **Spike S2:** author a minimal `calt` rule (`@HIGH_EXIT` → `.hi`), confirm it survives fontmake and fires in Chrome **and** in Word with `calt` on | 🤖 |
| 1.3 | **Spike S3:** read a `.glif` and assert an exit node's coordinate — proves the validator is feasible | 🤖 |
| 1.4 | Pin the Python environment (`requirements.txt`), install HarfBuzz binaries, document Windows setup | 🤖 |
| 1.5 | `git init`, `.gitignore` for `build/` and `proofs/`, first commit of the Phase 0 docs | 🤖 |
| 1.6 | Scaffold the repo structure per architecture §1 | 🤖 |
| 1.7 | **Decision gate:** spikes pass → stay on Fontra. Any spike fails → ⚠️ escalate the $49 FontCreator Essentials fallback | ⚠️🤝 |

**Exit criteria:** a three-letter word renders connected, from UFO source, via a one-command build.
If this doesn't hold, nothing downstream is worth starting.

---

## Phase 2 — Template & drawing round 1 🎨
**Epic 2 · Get the owner's hand onto paper under the geometric contract.** Weeks 1–2.

| ID | Story | Who |
|---|---|---|
| 2.1 | ⚠️ **Collect handwriting samples** from the owner | ⚠️🎨 |
| 2.2 | Measure x-height, cap height, ascender, descender, slant angle and natural join heights from the samples; fill in real values in `joins.yaml` | 🤖 |
| 2.3 | ⚠️ Owner approves the metrics and the two join heights | ⚠️🤝 |
| 2.4 | Build `gen_template.py`; generate `template-lowercase.svg` with guides, dashed join lines, edge tick marks, slant guides and `S01-x` labels | 🤖 |
| 2.5 | Print, test-drive one sheet, adjust box size and tick visibility | 🤝 |
| 2.6 | **Draw all 26 lowercase letters** on the template, respecting the tick-mark rule | 🎨 |
| 2.7 | Flag the known problem children while drawing: `b o v w` high exits, and `e` | 🎨 |

**Exit criteria:** 26 lowercase letters on paper, every entry and exit stroke crossing its box
border exactly at a tick.

---

## Phase 3 — Digitization 🎨🤖
**Epic 3 · Paper to UFO.** Weeks 2–3.

| ID | Story | Who |
|---|---|---|
| 3.1 | Scan sheet `S01` at 600 dpi grayscale, automatic adjustments off; archive to `scans/S01/` | 🎨 |
| 3.2 | Crop per-glyph background images, place into the UFO at known scale | 🤖 |
| 3.3 | **Trace all 26 letters** in Fontra over the background images | 🎨 |
| 3.4 | Build `validate_joins.py` (entry/exit coordinates, uniform sidebearings, node-count ceiling) | 🤖 |
| 3.5 | Run the validator; owner fixes flagged glyphs; repeat until clean | 🤝 |
| 3.6 | Cleanup pass to the architecture §8 standard — extrema, no overlaps, consistent terminals | 🎨 |
| 3.7 | Time one redraw→rescan→rebuild cycle (success criterion S6) | 🤝 |

**Exit criteria:** 26 glyphs in the UFO, all passing the validator.

---

## Phase 4 — Default joins 🤖
**Epic 4 · Make all 676 work, with no feature code.** Week 3.

| ID | Story | Who |
|---|---|---|
| 4.1 | Build the font; render every one of the 676 bigrams via `hb-view` | 🤖 |
| 4.2 | Assemble the full 676-cell proof sheet as a reviewable browser page | 🤖 |
| 4.3 | Draw the 26 `.hi` entry variants | 🎨 |
| 4.4 | Author `classes.fea` and the class-wide `@HIGH_EXIT` → `.hi` rule | 🤖 |
| 4.5 | `hb-shape` assertions: `on` shapes to `o n.hi`; spot-check the full high-exit matrix | 🤖 |
| 4.6 | **Tier A verification:** all 676 connect in Chrome *and* in Word with `calt` off | 🤝 |

**Exit criteria:** Tier A passes. At this point the cursive font is *functionally complete* —
everything after this is polish.

---

## Phase 5 — Tracker & proofs 🤖🎨
**Epic 5 · Make exception coverage legible.** Week 4.

| ID | Story | Who |
|---|---|---|
| 5.1 | Generate `bigrams.csv` (676 rows) and `boundaries.csv` (52) per the architecture §6 schema | 🤖 |
| 5.2 | Build `coverage.py` — counts by status, reports cells **reviewed** | 🤖 |
| 5.3 | Build `gen_proofs.py` — proof sheets keyed to tracker row IDs | 🤖 |
| 5.4 | **Owner reviews all 676 cells**, marking each `ok-default` or `flagged` | 🎨 |
| 5.5 | Triage the flagged set: assign mechanism (`calt` / `liga`) and exception glyph names | 🤝 |
| 5.6 | ⚠️ Owner decides which flagged cells are actually worth fixing | ⚠️🎨 |

**Exit criteria:** all 728 cells reviewed. A flagged list exists, sized by the owner's appetite —
not by the tracker.

**Scope guard:** this is where the project is most likely to balloon. The flagged list is a menu,
not a backlog. Timebox 5.4 to a single sitting.

---

## Phase 6 — Exceptions 🎨🤖
**Epic 6 · Optional polish. First to be cut (D-16, CUT-3).** Week 5.

| ID | Story | Who |
|---|---|---|
| 6.1 | Build `gen_features.py` — tracker → `cursive-calt.fea` + `cursive-liga.fea`, correctly ordered | 🤖 |
| 6.2 | Draw the approved exception forms | 🎨 |
| 6.3 | Regenerate features, rebuild, re-proof the affected cells | 🤖 |
| 6.4 | `hb-shape` assertions per shipped exception | 🤖 |
| 6.5 | Advance tracker rows to `exception-shipped` | 🤖 |
| 6.6 | French accents `é è ê à ç` as components | 🤝 |

**Exit criteria:** every approved exception ships and is machine-verified. Zero exceptions is a
valid outcome.

---

## Phase 7 — Simple font 🎨🤖
**Epic 7 · Font 2. The schedule buffer (D-16, CUT-5).** Week 5–6.

| ID | Story | Who |
|---|---|---|
| 7.1 | Generate the uppercase + lowercase template sheet | 🤖 |
| 7.2 | **Draw `A`–`Z` and `a`–`z`**, capitals harmonised with the cursive in weight and slant | 🎨 |
| 7.3 | Scan, trace, clean up | 🎨 |
| 7.4 | Draw `0`–`9` and punctuation | 🎨 |
| 7.5 | Conventional spacing and kerning — unconstrained by joins | 🤝 |
| 7.6 | **Port capitals, digits and punctuation into Font 1** (D-13) | 🤖 |
| 7.7 | Build Font 2; verify no `.notdef` boxes in either font | 🤖 |

**Exit criteria:** Font 2 builds and installs; Font 1 no longer renders boxes for capitals or
digits.

---

## Phase 8 — Polish, verification, documentation 🤝
**Epic 8 · Ship it.** Weeks 6–7.

| ID | Story | Who |
|---|---|---|
| 8.1 | Font metadata: ⚠️ family names, style, version, designer, licence | ⚠️🤝 |
| 8.2 | One-command build producing OTF + TTF + WOFF2 for both fonts | 🤖 |
| 8.3 | **Tier B verification:** exceptions fire in Chrome, Figma, Adobe, Word with `calt` on | 🤝 |
| 8.4 | **Tier C verification:** accents render; WOFF2 loads in Chrome, Firefox, Safari | 🤝 |
| 8.5 | Windows install test, both fonts | 🎨 |
| 8.6 | Final spacing pass via stub length — **never** via sidebearings or kerning (D-15) | 🎨 |
| 8.7 | ⚠️ **Owner reads a paragraph of set text and confirms it reads as their handwriting** (criterion S5) | ⚠️🎨 |
| 8.8 | Write `docs/paper-process.md` — the operator cheat sheet, derived from architecture §8 | 🤖 |
| 8.9 | Final journal entry and coverage report; retrospective | 🤝 |

**Exit criteria:** all PRD §5 Tier A and B checks pass; owner signs off on S5.

---

## Critical path and buffer

```
Epic 1 ──> Epic 2 ──> Epic 3 ──> Epic 4   ← critical path; the font is usable at the end of Epic 4
                                   │
                                   ├──> Epic 5 ──> Epic 6   ← polish, cuttable
                                   └──> Epic 7              ← buffer, cuttable entirely
                                                 │
                                                 └──> Epic 8
```

**The schedule insight:** Font 1 is functionally complete at the end of **Epic 4**, around week 3.
Epics 5–7 are all optional or cuttable. That is deliberate — it is the only structure in which a
7-week target for a connecting cursive is honest rather than hopeful.

## Owner time, concentrated

The owner is needed at the drawing table in four places: **2.6** (26 cursive letters), **3.3**
(tracing — the biggest single time sink and the steepest learning curve), **5.4** (676-cell proof
review, one sitting), and **7.2** (Font 2 letters). Everything else is agent-side.

## First Phase 1 story

**Story 1.1 — Spike S1.** Install Fontra Pak, create a UFO, draw three crude placeholder letters
(`a`, `b`, `n`), build with fontmake, shape with `hb-shape`, confirm a three-letter word renders
connected. Nothing else starts until this round-trips.
