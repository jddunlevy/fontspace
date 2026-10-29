# Environment Setup — fontspace

Windows. Reproduces the toolchain from scratch. Verified 2026-10-29.

Rationale for these choices is in `planning-artifacts/research/research-report.md` §Q1 and
`planning-artifacts/decision-log.md` D-03.

---

## What's installed

| Component | Version | Location | Tracked in git? |
|---|---|---|---|
| Python | 3.14.0 | `C:\Python314` | n/a (system) |
| Virtual env | — | `.venv/` | no (ignored) |
| fontmake | 3.12.2 | `.venv/Scripts/fontmake.exe` | pinned in `requirements.txt` |
| fonttools | 4.68.0 | `.venv/Lib/site-packages/` | pinned |
| brotli | 1.2.1 | `.venv/Lib/site-packages/` | pinned (required for WOFF2) |
| HarfBuzz CLI | 15.0.0 | `vendor/harfbuzz/` | no (ignored — see below) |
| Fontra Pak | — | `%LOCALAPPDATA%\Programs\Fontra Pak` | n/a (GUI app) |

Total spend: **$0**.

---

## 1. Python virtual environment

```powershell
cd C:\dev\fontspace
py -3.14 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
```

Use `py -3.14` (the `C:\Python314` install), **not** `python3` — that resolves to the Microsoft
Store Python 3.13, whose path redirection causes trouble for font tooling.

`brotli` is installed explicitly. It is not a hard fontmake dependency, but WOFF2 export does
not work without it.

To regenerate the pin after adding a package: `pip freeze > requirements.txt`.

**Scripts should call `.\.venv\Scripts\<tool>.exe` directly** rather than relying on an activated
shell, so builds work the same from an agent, a task runner, or a bare terminal.

---

## 2. HarfBuzz command-line tools

Not available via pip — the pip package `uharfbuzz` is the Python *binding*, not these binaries.

1. Download `harfbuzz-win64-<version>.zip` from
   <https://github.com/harfbuzz/harfbuzz/releases>
2. Extract and place the contents of `harfbuzz-win64/` directly into `vendor/harfbuzz/`

The `.exe` files **must** stay alongside their sibling `.dll` files — `hb-view` needs the cairo
DLLs to rasterise, and will fail silently if they're separated.

```powershell
# verify
.\vendor\harfbuzz\hb-shape.exe --version
.\vendor\harfbuzz\hb-view.exe --version
```

`vendor/` is gitignored: these are third-party binaries, re-downloadable from a pinned version.
Committing them would bloat the repo for no benefit.

### What these two tools are for

- **`hb-shape`** — prints the glyph IDs and advances a string shapes to. This is how join
  substitutions get *asserted* rather than eyeballed. Example, using a known Calibri ligature to
  show the shape of the output:

  ```
  > hb-shape.exe C:\Windows\Fonts\calibri.ttf "fi"
  [gid302=0+1084]            # two characters collapsed to ONE glyph: a ligature fired

  > hb-shape.exe C:\Windows\Fonts\calibri.ttf "on"
  [gid381=0+1080|gid374=1+1076]   # two glyphs: no substitution
  ```

  For this project the key assertion will be that `on` shapes to `o` + `n.hi`, not `o` + `n`.
  `--features=-calt` disables contextual alternates, which is how we reproduce Word's default
  behaviour on the command line (see PRD V-02).

- **`hb-view`** — renders shaped text to PNG/SVG/PDF. Used to generate deterministic proof
  images of all 676 bigrams that can be diffed between builds.

---

## 3. Fontra Pak

Download from <https://github.com/fontra/fontra-pak/releases> and install. It runs a local
server and opens the editor in your default browser.

Fontra reads and writes UFO and designspace **natively**, which is why it was chosen — the files
in `sources/` are the real source of truth, not an export from a proprietary project file.

---

## 4. Git

```powershell
git init
git config core.autocrlf false
```

`core.autocrlf false` is **not optional**. Git's Windows default rewrites line endings on
checkout, which would make every `.glif` and `.fea` file show spurious diffs — defeating the
entire reason text-based sources were chosen.

See `.gitignore` for what is and isn't tracked. Note that `scans/` **is** tracked: scans are the
primary process record and cannot be regenerated. If repo size becomes a problem, move them to
Git LFS rather than ignoring them.

---

## Verifying the whole stack

```powershell
.\.venv\Scripts\fontmake.exe --help          # build tool present
.\vendor\harfbuzz\hb-shape.exe --version     # shaping tool present
.\vendor\harfbuzz\hb-view.exe --version      # rasteriser present
```

The real end-to-end check needs a UFO to compile, and is spike **S1** (roadmap story 1.1):
draw three crude letters, build with fontmake, shape with `hb-shape`, confirm a three-letter
word renders connected.
