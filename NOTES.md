# Notes — R/Seurat ↔ Python/Scanpy

Quick reference for when muscle memory reaches for the wrong syntax.

## The object

| Concept | Seurat (R) | Scanpy (Python / AnnData) |
|---|---|---|
| The object | `seurat_obj` | `adata` |
| Counts matrix | `seurat_obj@assays$RNA@counts` | `adata.X` (or `adata.layers["counts"]` if we've saved a copy) |
| Normalised / log1p matrix | `seurat_obj@assays$RNA@data` | `adata.X` (after `normalize_total` + `log1p`) |
| Cell metadata | `seurat_obj@meta.data` | `adata.obs` (a pandas DataFrame) |
| Gene metadata | `seurat_obj@assays$RNA@meta.features` | `adata.var` |
| PCA / UMAP embeddings | `seurat_obj@reductions$pca@cell.embeddings` | `adata.obsm["X_pca"]`, `adata.obsm["X_umap"]` |
| Cluster IDs | `Idents(seurat_obj)` | `adata.obs["leiden"]` |
| "Raw" snapshot of the data | `seurat_obj@assays$RNA@data` (always-on) | `adata.raw` (only if you set it: `adata.raw = adata`) |

## Common operations

| Goal | R / Seurat | Python / Scanpy |
|---|---|---|
| Load 10X data | `Read10X("path/")` | `sc.read_10x_mtx("path/")` |
| Filter cells by gene count | `subset(obj, subset = nFeature_RNA > 200)` | `sc.pp.filter_cells(adata, min_genes=200)` |
| Filter genes | `subset(obj, features = rownames(obj)[Matrix::rowSums(obj) > 3])` | `sc.pp.filter_genes(adata, min_cells=3)` |
| Normalise + log1p | `NormalizeData(obj)` | `sc.pp.normalize_total(adata, target_sum=1e4); sc.pp.log1p(adata)` |
| Find variable genes | `FindVariableFeatures(obj)` | `sc.pp.highly_variable_genes(adata, n_top_genes=2000, flavor="seurat")` |
| Scale | `ScaleData(obj)` | `sc.pp.scale(adata, max_value=10)` |
| PCA | `RunPCA(obj)` | `sc.tl.pca(adata)` |
| KNN graph | `FindNeighbors(obj)` | `sc.pp.neighbors(adata)` |
| Clustering | `FindClusters(obj, resolution=0.5)` | `sc.tl.leiden(adata, resolution=0.5)` |
| UMAP | `RunUMAP(obj)` | `sc.tl.umap(adata)` |
| Marker genes per cluster | `FindAllMarkers(obj)` | `sc.tl.rank_genes_groups(adata, groupby="leiden", method="wilcoxon")` |
| UMAP plot | `DimPlot(obj)` | `sc.pl.umap(adata, color="leiden")` |
| Feature plot | `FeaturePlot(obj, "CD14")` | `sc.pl.umap(adata, color="CD14")` |
| Violin plot | `VlnPlot(obj, "CD14")` | `sc.pl.violin(adata, "CD14", groupby="leiden")` |
| Save object | `saveRDS(obj, "obj.rds")` | `adata.write_h5ad("obj.h5ad")` |
| Load object | `readRDS("obj.rds")` | `sc.read_h5ad("obj.h5ad")` |

## Syntax gotchas for an R user

- **Indexing:** Python is 0-indexed. `adata[0:10, :]` is the first 10 cells; in R `obj[1:10, ]` is the first 10.
- **`.copy()`:** when you slice (`adata[adata.obs["pct_mt"] < 20, :]`), Scanpy returns a *view*, not a new object. Re-assignments to a view warn. Always chain `.copy()` when filtering: `adata = adata[mask, :].copy()`.
- **DataFrame access:** `adata.obs["celltype"]` is a column (like `obj@meta.data$celltype`). `adata.obs.loc[mask]` is row-filtering. `adata.obs["newcol"] = values` adds a column.
- **Categorical columns:** Scanpy expects factor-like columns as `pd.Categorical`. If you build one from a list: `adata.obs["ct"] = pd.Categorical(values)`.
- **In-place vs return:** most `sc.pp.*` and `sc.tl.*` functions modify `adata` **in place** and return `None`. Do NOT write `adata = sc.pp.normalize_total(adata, ...)` — that sets `adata = None`.
- **Chaining:** Scanpy doesn't pipe. Write each step on its own line.

## Where things live in this repo

| What | Where |
|---|---|
| The main notebook | `notebooks/01_smillie_ibd_pipeline.ipynb` |
| Reference rendered output | `notebooks/01_smillie_ibd_pipeline.html` (open in browser) |
| Raw data | `data/` (gitignored; download separately — see README) |
| Processed AnnData | `data/processed/*.h5ad` (written by the notebook) |
| Conda env spec | `environment.yml` |

## When something goes wrong

1. **Kernel dies or a cell errors** — scroll up through the traceback, Python traceback errors are verbose but the line number points to the actual line.
2. **Memory error on Smillie full data** — subset to 10–20k cells with `sc.pp.subsample(adata, n_obs=20000)` and re-run.
3. **`ModuleNotFoundError`** — the env is wrong or not activated. Check with `which python` (should point inside `miniforge3/envs/scanpy/`).
4. **Plot doesn't show in Jupyter** — add `plt.show()` at the end of the cell or `%matplotlib inline` at the top of the notebook.
