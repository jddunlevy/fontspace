# Architecture — fontspace

Date: 2026-10-29 · Phase 0 design document. Nothing here is built yet.

Covers repo structure, source formats, toolchain, joining model, glyph naming, tracker schema,
build/test pipeline, the paper process and template-sheet spec, and the division of labour.

---

## 1. Repo structure

```
fontspace/
├── bmad/                          # BMAD config
├── planning-artifacts/            # this phase's docs
├── implementation-artifacts/
│   └── stories/                   # story files, Phase 1+
├── docs/
│   ├── journal.md                 # dated log of drawing rounds + decisions
│   └── paper-process.md           # operator-facing cheat sheet (derived from §8)
├── sources/
│   ├── Cursive.designspace
│   ├── Cursive.ufo/               # UFO3 — the source of truth for Font 1
│   │   ├── glyphs/                # one .glif per glyph (text, diffable)
│   │   ├── images/                # scan crops used as tracing backgrounds
│   │   ├── features.fea           # assembled: includes the files below
│   │   ├── fontinfo.plist
│   │   └── lib.plist
│   ├── Simple.designspace
│   ├── Simple.ufo/
│   ├── features/
│   │   ├── cursive-calt.fea       # GENERATED from the tracker — do not hand-edit
│   │   ├── cursive-liga.fea       # GENERATED from the tracker — do not hand-edit
│   │   ├── classes.fea            # hand-authored: @HIGH_EXIT, @LOWERCASE, etc.
│   │   └── manual.fea             # hand-authored escape hatch
│   └── data/
│       ├── bigrams.csv            # 676 rows — the tracker
│       ├── boundaries.csv         # 52 rows
│       └── joins.yaml             # canonical join-height + metrics constants
├── scans/
│   └── S01/                       # raw 600dpi scans, by sheet ID
├── templates/
│   └── template-lowercase.svg     # printable drawing sheet (generated from joins.yaml)
├── tools/                         # Phase 2+ scripts
│   ├── gen_features.py            # tracker  -> .fea
│   ├── gen_proofs.py              # tracker  -> proof sheets (all 676)
│   ├── coverage.py                # tracker  -> coverage report
│   ├── validate_joins.py          # UFO      -> pass/fail per glyph
│   └── gen_template.py            # joins.yaml -> template SVG
├── proofs/                        # generated output, gitignored
├── build/                         # generated binaries, gitignored
└── requirements.txt               # pinned Python env
```

**Rule:** anything under `sources/` is truth. Anything in `proofs/` or `build/` is disposable
output. Generated `.fea` files carry a do-not-edit header; edits belong in the tracker.

---

## 2. Source formats

| Artifact | Format | Why |
|---|---|---|
| Glyph outlines | **UFO3** (`.ufo`, one `.glif` per glyph) | Text-based, one file per glyph, clean git diffs, agent-writable, Fontra-native |
| Multi-source wiring | `.designspace` | Standard, future-proofs a second weight without restructuring |
| OpenType features | `.fea` (AFDKO syntax) | Text, generatable, the lingua franca of font build tools |
| Tracker | **CSV** | Diffs per row; readable without tooling; parses with the stdlib |
| Constants | **YAML** (`joins.yaml`) | One place for join heights and metrics; consumed by template generator, validator and proofs |
| Scans | PNG, 600 dpi grayscale | Lossless, no JPEG artefacts at stroke edges |

**Why CSV for the tracker, not YAML/JSON/SQLite:** 676 rows of mostly-identical records. CSV
gives a one-line-per-cell diff, so a review session shows up in git as exactly the rows that
changed. YAML would be 5× the lines; JSON reformats noisily; SQLite is a binary blob and
defeats review.

---

## 3. Toolchain

| Role | Tool | Cost |
|---|---|---|
| Glyph drawing / tracing | **Fontra Pak** (Windows, UFO-native, background images) | $0 |
| Build | **fontmake** (+ ufo2ft) via pip, Python ≥ 3.10 | $0 |
| WOFF2 | **fonttools** `ttLib.woff2` | $0 |
| Shaping verification | **HarfBuzz** `hb-shape` (assertions), `hb-view` (raster proofs) | $0 |
| Visual proofing | Local HTML proof page in Chrome, plus generated proof sheets | $0 |
| Scan prep | Windows Scan / vendor driver; GIMP for levels if needed | $0 |
| Reference (optional) | Inkscape, auto-trace underlay only | $0 |
| **Escape hatch** | FontCreator 16 Essentials, if the Fontra spike fails | $49 |

**Total planned spend: $0**, against a $100 ceiling. Rationale and rejected alternatives are in
`research/research-report.md` §Q1.

Environment is pinned in `requirements.txt` and installed into a venv, so builds are
reproducible and agent runs don't drift.

---

## 4. Joining model

### Constants (`sources/data/joins.yaml`)

Values are placeholders until handwriting samples are measured; the structure is fixed.

```yaml
units_per_em: 1000
x_height: 500          # measured from samples
cap_height: 700
ascender: 750
descender: -250
slant_angle: 12        # degrees, measured from samples

join_low:  220         # ~44% of x-height — the default join
join_high: 500         # ~x-height — exits of b, o, v, w

stub_length: 60        # horizontal run of entry and exit stubs
sidebearing: 0         # fixed for all connecting glyphs; stubs carry the spacing
```

### The geometric contract

Every connecting lowercase glyph must satisfy:

1. Its **entry stub** starts at exactly `(0, entry_height)`.
2. Its **exit stub** ends at exactly `(advance_width, exit_height)`.
3. Both stubs run horizontally for `stub_length` before curving into the letter body.
4. `sidebearing` is identical across all connecting glyphs.

Because entry and exit heights come from a two-value set and every glyph terminates on the exact
coordinate, **any letter can follow any letter and the strokes meet.** No lookup is involved.
This is what makes all 676 bigrams work by default, and what makes the font survive in Word with
contextual alternates disabled.

### The two heights

| Class | Letters | Exit height |
|---|---|---|
| Low exit (default) | `a c d e f g h i j k l m n p q r s t u x y z` | `join_low` |
| **High exit** | `b o v w` | `join_high` |

Entry heights mirror this: every letter has a default entry at `join_low`, and a `.hi` variant
entering at `join_high` for use after a high-exit letter.

### Feature code

```fea
# classes.fea
@HIGH_EXIT  = [b o v w];
@LOWERCASE  = [a b c d e f g h i j k l m n o p q r s t u v w x y z];
@LC_DEFAULT = [a b c d e f g h i j k l m n o p q r s t u v w x y z];
@LC_HI      = [a.hi b.hi c.hi d.hi e.hi f.hi g.hi h.hi i.hi j.hi k.hi l.hi
               m.hi n.hi o.hi p.hi q.hi r.hi s.hi t.hi u.hi v.hi w.hi x.hi y.hi z.hi];

# cursive-calt.fea  (GENERATED)
feature calt {
    # High-entry variant after a high-exiting letter
    sub @HIGH_EXIT @LC_DEFAULT' by @LC_HI;
    # Per-pair exceptions appended here from the tracker
} calt;
```

Lookup ordering matters: the class-wide high-entry rule runs first, then specific per-pair
exceptions, so an exception can override the general rule. The generator emits them in that
order.

### Rejected alternatives

- **GPOS `curs`** — HarfBuzz honours it, but DirectWrite/Word and Adobe are unreliable for Latin,
  and it only translates glyphs; it cannot reshape a badly-angled exit.
- **`rclt`** — no advantage over `calt` here, worse app support.
- **Three or more join heights** — multiplies alternate sets and review burden past what the
  timeline absorbs.
- **Separate stroke glyphs** (entry/core/exit as independent glyphs) — most flexible model, far
  too much machinery for seven weeks.
- **Single join height** — produces the documented `b o v w` dipping artefact.

---

## 5. Glyph naming convention

| Pattern | Meaning | Example |
|---|---|---|
| `a` … `z` | base lowercase, default (low) entry | `n` |
| `A` … `Z` | capitals, non-connecting | `N` |
| `zero` … `nine` | digits (AGLFN names, not literals) | `four` |
| `<base>.hi` | high-entry variant, for use after `b o v w` | `n.hi` |
| `<base>.init` / `.fina` | word-positional variants (stretch) | `n.init` |
| `<first>_<second>.calt` | per-pair contextual exception form | `o_n.calt` |
| `<first>_<second>.liga` | true ligature, two letters as one drawing | `t_t.liga` |
| `<base><accent>` | precomposed accented glyph (AGLFN) | `eacute`, `ccedilla` |
| `.notdef` | missing-glyph box | — |

Rules: AGLFN names throughout (never literal characters in filenames), suffixes are lowercase,
only one suffix level deep, and ligature/exception components are joined with `_` in reading
order.

---

## 6. Combination tracker schema

### `sources/data/bigrams.csv` — 676 rows

| Column | Type | Notes |
|---|---|---|
| `id` | string | `<first>_<second>`, e.g. `o_n`. Primary key |
| `first` | string | glyph name of the first letter |
| `second` | string | glyph name of the second letter |
| `first_exit` | enum | `low` \| `high` — derived from `first`, stored for sortability |
| `second_entry` | enum | `low` \| `high` — which entry form the default rules select |
| `status` | enum | see lifecycle below |
| `mechanism` | enum | `none` \| `calt` \| `liga` |
| `exception_glyph` | string | glyph name, or empty |
| `sheet_id` | string | paper sheet the source letters came from, e.g. `S01` |
| `reviewed_on` | date | ISO date, or empty |
| `notes` | string | free text; why it was flagged |

### `sources/data/boundaries.csv` — 52 rows

Same shape, with `position` (`initial` \| `final`) replacing `first`/`second` pairing, and a
single `letter` column.

### Status lifecycle

```
unreviewed
    ├──> ok-default          # looked at it; the geometric default is fine. TERMINAL, and the
    │                        # expected destination for the large majority of the 676
    └──> flagged             # looks wrong or weak; worth an exception
            ├──> exception-planned   # mechanism + glyph name decided
            │        └──> exception-drawn    # alternate form exists in the UFO
            │                 └──> exception-shipped  # generated into .fea and verified
            └──> wontfix     # flagged, but deliberately not fixing. TERMINAL
```

### Framing — important

The tracker records **potential** exceptions. It is **not** a to-do list, because the geometric
defaults already cover all 676 combinations. `ok-default` is a success state, not an unfinished
one. Therefore:

- The coverage report measures **cells reviewed**, never "cells completed."
- A tracker that is 100% `ok-default` describes a finished, correct font.
- `flagged` counts are an aesthetic signal, not a backlog burn-down.

### Combinatorics, stated explicitly

| Set | Count | In scope |
|---|---|---|
| Ordered lowercase bigrams (26 × 26) | **676** | Yes — all reviewed at least once |
| Word-boundary positions (26 initial + 26 final) | **52** | Yes |
| **Total review cells** | **728** | Yes |
| Trigrams (26³) | 17,576 | **No** — only by individual flag |
| Bigrams involving capitals, digits or punctuation | — | **No** — those glyphs don't connect |

---

## 7. Build and test pipeline

### Build

```
sources/*.ufo  +  sources/features/*.fea
        │
        ├─ gen_features.py   (tracker -> cursive-calt.fea, cursive-liga.fea)
        │
        ▼
    fontmake  ──>  build/*.otf, build/*.ttf
        │
        ▼
    fonttools woff2  ──>  build/*.woff2
```

One entry point (`build.py` or a `make`-equivalent) runs: generate features → fontmake → WOFF2 →
validator → proofs.

### Test layers

| Layer | Tool | Checks |
|---|---|---|
| **Source validation** | `validate_joins.py` | Every connecting glyph's entry node is at `(0, entry_height)` and exit node at `(advance_width, exit_height)`; sidebearings uniform; node count under ceiling. Runs against UFO, pre-build |
| **Shaping assertions** | `hb-shape` | Typing `on` yields `o n.hi`, not `o n`. Machine-verified substitution, per tracker row with a mechanism |
| **Raster proofs** | `hb-view` | Deterministic PNG of every bigram for diffing between builds |
| **Visual proof sheet** | `gen_proofs.py` + browser page | All 676 on a reviewable grid; the human gate |
| **Environment matrix** | manual | Chrome, Figma, Adobe, Word (`calt` on *and* off), Windows install. Tier A/B/C per PRD §5 |

The validator is the keystone. Hand-drawn joins landing off-coordinate is risk #1, and it is the
one risk that is cheap to eliminate automatically and expensive to catch by eye.

---

## 8. Paper process and template-sheet spec

### Template sheet (`templates/template-lowercase.svg`, generated from `joins.yaml`)

Printed at a **known physical size** so scan scale is deterministic — no guessing at import.

Each glyph box contains:

| Element | Rendering | Purpose |
|---|---|---|
| Box border | thin solid, light grey | glyph bounds = advance width |
| Baseline | solid | sit letters on it |
| x-height | solid, lighter | body height |
| Cap height / ascender / descender | dotted | reference only |
| **`join_low` line** | **dashed, accent colour** | where low entries and exits must cross |
| **`join_high` line** | **dashed, second accent colour** | where `b o v w` exits must cross |
| **Edge tick marks** | short solid ticks on left and right borders, at both join heights | **the hard constraint — strokes must cross the border exactly at a tick** |
| Slant guides | faint diagonals at `slant_angle` | keeps the hand consistent |
| Label | small, bottom-left of each box: `S01-a` | ties scan ↔ glyph ↔ tracker `sheet_id` |

Print in a light non-photo colour so guides are easy to drop out during scan cleanup.

**The one rule that matters:** a letter is valid only if its entry stroke crosses the left border
exactly at its entry tick, and its exit stroke crosses the right border exactly at its exit tick.
Everything else about the letter is free. This single constraint is what makes hand-drawn letters
align digitally.

### Scanning

| Setting | Value | Why |
|---|---|---|
| Resolution | **600 dpi** | enough detail for stroke edges without unwieldy files |
| Mode | **grayscale** | bitonal destroys the soft stroke edges needed for faithful tracing |
| Auto-contrast / auto-level | **off** | keeps scans comparable across sessions |
| Auto-crop / deskew | **off** | preserves the known physical scale and registration |
| Format | PNG | lossless at stroke edges |
| Filing | `scans/S01/S01.png` plus per-glyph crops | one folder per sheet ID |

### Tracing

**Manual tracing in Fontra, over the scan as a glyph background image.**

Auto-trace is **rejected as the primary path**: Inkscape Trace Bitmap on inked pen texture
produces outlines with hundreds of nodes and ragged edges, which is fatal when joins must land on
an exact coordinate and stub shapes must match across 26 glyphs. Auto-trace remains useful as a
read-only underlay for checking proportions.

### Cleanup standard

1. Node count at or under the ceiling (target ≤ 30 per lowercase glyph; the validator enforces).
2. Points on extrema (topmost, bottommost, leftmost, rightmost of each curve).
3. No overlapping contours within a glyph.
4. Consistent terminal shapes across all glyphs — the stubs are a family trait, not per-letter.
5. **Hard rule:** entry node exactly `(0, entry_height)`; exit node exactly
   `(advance_width, exit_height)`.
6. Stub geometry matches `stub_length` and leaves the border horizontally, so joins meet tangent
   rather than at a corner.

### Documentation

- `docs/journal.md` — dated entries per drawing round, per decision, per spike result. Terse.
- `planning-artifacts/decision-log.md` — the durable record of decisions and rejected alternatives.
- `scans/` — the primary process evidence; the scans *are* the record. No write-up beyond the
  journal.

---

## 9. Division of labour

| Area | Owner (Jaxon) | Agents |
|---|---|---|
| Aesthetic direction, slant, weight, loop style | **decides** | advise only |
| Drawing letters on paper | **does all of it** | — |
| Scanning | does | document settings |
| Tracing and outline cleanup | does (with a learning curve) | build the validator that checks the work |
| Metrics / join constants | approves | measure from samples, propose values |
| Template sheet | approves | generate from `joins.yaml` |
| Glyph naming, UFO structure | — | own it |
| Feature code (`.fea`) | — | own it, generated from the tracker |
| Tracker maintenance | reviews proof sheets, sets `ok-default` / `flagged` | build it, generate rows, report coverage |
| Build pipeline, WOFF2, CI | — | own it |
| Proof generation, shaping assertions | reviews output | own it |
| Environment matrix testing | does the manual app checks | automate what's automatable |
| Exception decisions | **decides which flagged cells get fixed** | propose mechanism, draw nothing |

**Where the owner must be at the drawing table:** drawing rounds, tracing, proof review, and
every aesthetic call. Everything else is agent-side and can proceed in parallel.
