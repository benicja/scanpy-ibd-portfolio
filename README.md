# scanpy-ibd-portfolio

End-to-end Scanpy analysis of a public ulcerative colitis scRNA-seq dataset
([Smillie et al. 2019, *Cell*](https://doi.org/10.1016/j.cell.2019.06.029);
Broad Single Cell Portal study [SCP259](https://singlecell.broadinstitute.org/single_cell/study/SCP259)).

A small, self-contained portfolio project: QC → normalisation → HVG → PCA →
neighbours/UMAP → Leiden clustering → marker genes → manual cell-type annotation,
focused on the myeloid compartment of healthy vs inflamed colonic mucosa.

## Repository layout

```
notebooks/    Jupyter notebooks (the main deliverable)
data/         Raw + processed data (gitignored)
figures/      Exported figures
environment.yml   Conda environment spec
```

## Setup

### Option 1 — conda (recommended)

Requires conda. On Windows we recommend
[Miniforge](https://github.com/conda-forge/miniforge/releases) (lean installer,
pre-configured for the `conda-forge` channel).

**First-time install of Miniforge on Windows:**

1. Download
   [`Miniforge3-Windows-x86_64.exe`](https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Windows-x86_64.exe).
2. Run the installer. Accept defaults, except on the *Advanced Options* screen
   tick **Add Miniforge3 to my PATH environment variable** (off by default).
   Leave *Register Miniforge3 as the system Python* **unticked** if you already
   have another Python installation.
3. Close and reopen any terminals so `conda` becomes available on the PATH.
4. Verify with `conda --version`.

**macOS / Linux:** download the matching installer from the
[Miniforge releases page](https://github.com/conda-forge/miniforge/releases)
and run `bash Miniforge3-<os>-<arch>.sh`.

**Create the project environment:**

```
git clone https://github.com/benicja/scanpy-ibd-portfolio.git
cd scanpy-ibd-portfolio
conda env create -f environment.yml
conda activate scanpy
jupyter lab
```

First run takes ~10 minutes (downloads ~500 MB of packages). Subsequent
activations are instant.

### Option 2 — pip + venv (fallback)

If conda is unavailable, a plain virtualenv works too:

```
git clone https://github.com/benicja/scanpy-ibd-portfolio.git
cd scanpy-ibd-portfolio
python -m venv .venv
# Windows:  .venv\Scripts\activate
# macOS/Linux:  source .venv/bin/activate
pip install -r requirements.txt
jupyter lab
```

### Adding new packages

`environment.yml` is hand-curated — do **not** regenerate it with
`conda env export` (which dumps hundreds of platform-specific transitive
dependencies). When adding a new package, install it, then manually add one
line to both `environment.yml` and `requirements.txt`, and commit.

## Data

The Smillie dataset is not redistributed here. Download from the
[Broad Single Cell Portal, SCP259](https://singlecell.broadinstitute.org/single_cell/study/SCP259)
(free account required) and place the files under `data/smillie/`.
