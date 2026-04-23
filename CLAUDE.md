# CLAUDE.md — instructions for Claude Code working in this repo

## Who is using this repo

**Alicja A. Nowacka** — PhD Immunologist, University of Lübeck (graduating
March 2026). GitHub: `benicja`.

- Scientifically sophisticated: 8 years wet-lab immunology (complement
  biology, dendritic cell progenitors, cDC1/cDC2 fate decisions), 4+
  publications, flow cytometry, mouse models.
- **R + Seurat is her native computational language** (publication-level,
  self-taught, used for real scRNA-seq analyses during her PhD).
- **Zero prior Python / Scanpy experience.** Learning for the first time.

## Why this repo exists

Preparation for a first-stage interview at **Roche Institute of Human Biology
(IHB), Basel** for an Intestinal Inflammation PostDoc position.

- **Interview:** Monday 27 April 2026, 14:00 CEST with Tim Recaldin.
- **Target postdoc start:** September 2026.
- **Scientific context:** IHB uses human intestinal organoids and 10X
  scRNA-seq; the Intestinal Inflammation team cares about
  epithelial–immune crosstalk in IBD (UC, Crohn's), with strong myeloid
  interest — which aligns with Alicja's DC background.

This project is not a publication — it is **honest credibility prep**. The
goal is for her to truthfully say: *"I've been ramping up on Scanpy this
month, working through a public UC dataset — I'm comfortable with the
AnnData workflow and the conceptual pipeline."*

## How to collaborate with her

- **Translate Python concepts from R/Seurat equivalents** wherever possible.
  The cheat sheet in [`NOTES.md`](NOTES.md) covers the most common ones.
- **Do not assume Python syntax familiarity** — explain `.obs`, `.var`,
  `.X`, DataFrames, indexing, `.copy()` on views, in-place vs return.
- **Do NOT over-explain the biology or scRNA-seq concepts** themselves.
  She knows them better than you do.
- **Teach the *why*, not just the *how*** — every line of notebook code
  must be defensible by her in an interview. If you make a parameter
  choice, explain why; if there's a judgment call (Leiden resolution,
  cluster annotation, filter thresholds), flag it and let her decide.
- **Do not help her overstate her Python experience.** "Ramping up this
  month" is the honest framing; "comfortable" or "experienced" is not.
- **Match R/Seurat conventions when possible.** `flavor="seurat"` for HVGs,
  `method="wilcoxon"` for markers, etc.

## First-session setup (run these in order on a fresh machine)

On the very first run, after cloning this repo, Claude should walk Alicja
through the following. Each step is safe to run autonomously unless noted.

### 1. Verify / set git identity for this repo

Per-repo only — do NOT touch the global git config:

```
git config user.name "benicja"
git config user.email "260913572+benicja@users.noreply.github.com"
```

### 2. Verify GitHub CLI auth

Run `gh auth status`. If not logged in as `benicja`, run `gh auth login`
using device-flow web-browser auth and guide her through the one-time code
step. After auth, run `gh auth setup-git` so `git push` uses the GH token.

### 3. Check / install conda

Run `conda --version`. If missing:

- **Windows:** download
  `https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Windows-x86_64.exe`
  and install silently with:
  `<installer> /InstallationType=JustMe /RegisterPython=0 /AddToPath=1 /S /D=C:\Users\<user>\miniforge3`
- **macOS / Linux:** download the matching installer from
  https://github.com/conda-forge/miniforge/releases/latest and run with
  `bash <installer>.sh -b -p $HOME/miniforge3` then
  `$HOME/miniforge3/bin/conda init`.

After install, a new shell is needed for `conda` to be on PATH. From Claude
Code, use the full path (`<prefix>/Scripts/conda.exe` on Windows,
`<prefix>/bin/conda` elsewhere) until the env is activated.

### 4. Create the scanpy env

```
conda env create -f environment.yml
```

This takes ~10 min and downloads ~500 MB.

**Do NOT add `scrublet` back to `environment.yml` or `requirements.txt`.**
The notebook uses scanpy's built-in `sc.pp.scrublet` (requires
`scikit-image`, already included). The external `scrublet` pip package
pulls `annoy` which has no Windows py3.11 wheel and needs MSVC to compile.

### 5. Register a Jupyter kernel in the env

The notebook uses the default `python3` kernel name so no per-machine
registration is strictly required, but it's still cleanest to have the env
registered:

```
conda activate scanpy
python -m ipykernel install --user --name scanpy --display-name "Python (scanpy-ibd)"
```

### 6. Smoke-test the env

Launch the notebook at `notebooks/01_smillie_ibd_pipeline.ipynb`. It has
`USE_PBMC_FALLBACK = True` at the top, so it runs end-to-end on a built-in
2700-cell PBMC dataset in ~30–60 seconds, with no external data required.
Output should match `notebooks/01_smillie_ibd_pipeline.html` (reference
render).

### 7. Launch JupyterLab and hand over the URL

```
jupyter lab --no-browser --port 8888 --ServerApp.open_browser=False
```

Run in the background, wait for the `http://localhost:8888/lab?token=…` URL
to appear in the log, give that URL to Alicja.

## Where things are

- `notebooks/01_smillie_ibd_pipeline.ipynb` — the main (source) notebook
- `notebooks/01_smillie_ibd_pipeline.html` — executed reference render
- `NOTES.md` — R/Seurat ↔ Scanpy cheat sheet
- `environment.yml` / `requirements.txt` — dependency specs (hand-curated;
  do not regenerate with `conda env export`)
- `data/` — gitignored; Smillie dataset download goes in `data/smillie/`

## After setup: the real work

Once the PBMC3k smoke run succeeds, the next steps (in order) are:

1. Have Alicja pick a Leiden resolution from the three plotted (0.3 / 0.5 /
   0.8) and explain her reasoning — this is her first interview-defendable
   judgment call.
2. Walk through the marker-gene dot plot and fill in the
   `cluster_to_celltype` dictionary with her biological annotations.
3. Download the Smillie dataset from Broad Single Cell Portal study
   [SCP259](https://singlecell.broadinstitute.org/single_cell/study/SCP259)
   (she'll need to create a free account). Once downloaded to
   `data/smillie/`, flip `USE_PBMC_FALLBACK = False` and rerun.
4. Myeloid subcluster + annotate (cDC1 / cDC2 / mono / macrophage etc.) —
   this is the myeloid-focused story that matches the Roche IHB team's
   interests and Alicja's DC background.

Leave the judgment calls (filter thresholds, clustering resolution, cell
type assignments) to her — that's what makes the project defensible.
