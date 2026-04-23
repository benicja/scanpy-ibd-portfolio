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
[Miniforge](https://github.com/conda-forge/miniforge/releases) (installs a lean
conda pre-configured to use the `conda-forge` channel).

```
git clone https://github.com/benicja/scanpy-ibd-portfolio.git
cd scanpy-ibd-portfolio
conda env create -f environment.yml
conda activate scanpy
jupyter lab
```

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
