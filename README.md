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

Requires conda (Miniforge recommended on Windows). One-time:

```
conda env create -f environment.yml
conda activate scanpy
```

Then launch Jupyter:

```
jupyter lab
```

## Data

The Smillie dataset is not redistributed here. Download from the
[Broad Single Cell Portal, SCP259](https://singlecell.broadinstitute.org/single_cell/study/SCP259)
(free account required) and place the files under `data/smillie/`.
