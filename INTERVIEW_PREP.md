# 30-minute interview prep walkthrough

A timed, hands-on session designed to take you from *"I have a notebook
open"* to *"I can narrate this pipeline out loud."* Read with your partner
acting as prompter; you drive the keyboard.

**Goal:** by the end you should be able to (a) explain what an `AnnData`
object is in 30 seconds, (b) walk through the pipeline section by section
without notes, and (c) answer the three most likely interview questions.

This doc is reusable — come back to it the morning of the interview.

---

## Phase 1 — The AnnData mental model (8 min)

**Why this comes first:** if an interviewer asks you *"so what's an AnnData
object?"* and you fumble, you've lost the room. This is the cheapest, most
valuable knowledge in the whole project.

**Open a new cell at the very top of the notebook** (click the first cell,
press **B** to insert a new cell below). Paste:

```python
import scanpy as sc
adata = sc.datasets.pbmc3k()
adata
```

Run it (Shift+Enter). You should see a one-line summary like:

```
AnnData object with n_obs × n_vars = 2700 × 32738
    var: 'gene_ids'
```

### Try these one by one — say what you expect *before* running each

```python
adata.X            # the expression matrix
```
**Say out loud:** *"This is the equivalent of `obj@assays$RNA@counts` in
Seurat — a sparse matrix, cells × genes."*

```python
adata.obs.head()   # cell metadata
```
**Say:** *"This is `obj@meta.data` in Seurat — one row per cell. Right now
it's empty because we just loaded raw 10X data, but after QC this is where
`pct_counts_mt`, `n_genes_by_counts`, `leiden` cluster IDs all live."*

```python
adata.var.head()   # gene metadata
```
**Say:** *"One row per gene. After we run HVG, a `highly_variable` boolean
column appears here."*

```python
adata.shape        # (n_cells, n_genes)
```
**Say:** *"Cells first, genes second. Always."*

```python
adata.obs_names[:5]   # cell barcodes
adata.var_names[:5]   # gene symbols
```
**Say:** *"Cell barcodes are `obs_names`, gene symbols are `var_names`.
Same as `colnames()` and `rownames()` of a Seurat object's counts matrix."*

### The five-slot cheat (memorise this — you'll be asked)

| Slot | Holds | Seurat equivalent |
|---|---|---|
| `adata.X` | Expression matrix | `obj@assays$RNA@counts` / `@data` |
| `adata.obs` | Cell metadata DataFrame | `obj@meta.data` |
| `adata.var` | Gene metadata DataFrame | `obj@assays$RNA@meta.features` |
| `adata.obsm` | Embeddings — `X_pca`, `X_umap` | `obj@reductions` |
| `adata.uns` | Unstructured stuff | misc `@misc` |

**30-second answer to "what is AnnData?":**

> *"It's the central data structure in the Scanpy ecosystem — analogous
> to a SeuratObject. Cells × genes expression matrix in `.X`, per-cell
> metadata in `.obs`, per-gene metadata in `.var`, embeddings like PCA and
> UMAP in `.obsm`. Most Scanpy functions take an `adata` and modify it
> in place."*

Practice that sentence out loud once. Then move on.

---

## Phase 2 — Run the notebook + narrate (15 min)

**Now Run → Run All Cells.** Takes ~30–60 seconds. While it runs:

**Set the rule:** at every section heading, your partner stops you and
asks *"what does this step do, and why is it here?"* You answer in your
own words before scrolling further. This is the entire point of the
exercise — narration reveals gaps that reading hides.

If you can't answer a section, **flag it** (highlight the cell, add a
comment, or just say "skip — come back"). Don't fake it.

### Section-by-section prompts

For each section, here's what you should be able to say. Read these
*after* you've tried answering yourself — not before.

#### Section 3 — QC metrics
> *"I'm flagging mitochondrial and ribosomal genes by name prefix, then
> computing per-cell QC metrics: total counts, gene counts, and percent
> mitochondrial. High mito % means dying or stressed cells. This is the
> Python equivalent of `PercentageFeatureSet(obj, pattern='^MT-')`."*

**Likely question:** *"Why MT and not mt?"* → *"Human gene symbols are
uppercase MT-; mouse is lowercase mt-. Smillie is human."*

#### Section 4 — Filtering
> *"I'm dropping cells with fewer than 200 genes detected — likely empty
> droplets or dying cells. Dropping genes seen in fewer than 3 cells —
> noise. And capping mito % at 20% — colonic epithelium tolerates more
> mitochondrial reads than PBMCs, so this is more permissive than the 10%
> you'd use for blood."*

**Likely question:** *"Why those numbers?"* → *"They're standard 10X
first-pass thresholds — defensible from the QC violin plots above. Not
universal — a different tissue would justify different cuts."*

#### Section 5 — Doublet detection
> *"Scrublet generates synthetic doublets by averaging random cell pairs,
> scores each real cell against that distribution, and flags outliers.
> Doublets are a within-droplet artefact, so ideally run per-sample, not
> on pooled data."*

**Why scanpy's built-in:** *"I'm using `sc.pp.scrublet` (scanpy's built-in
implementation) rather than the external scrublet package — same algorithm,
no Windows C++ dependency."*

#### Section 6 — Normalisation + log1p
> *"Two-step normalisation, equivalent to Seurat's `NormalizeData()`:
> scale each cell to 10,000 total counts to correct sequencing depth,
> then log1p so highly-expressed genes don't dominate variance.
> I save the raw counts in `.layers['counts']` first — useful later
> for differential expression."*

#### Section 7 — Highly variable genes
> *"Restrict downstream analysis to the 2000 most variable genes — removes
> housekeeping noise. `flavor='seurat'` gives the same binned mean/dispersion
> selection as `FindVariableFeatures(selection.method='vst')`. I freeze
> the full matrix in `.raw` so I can still plot any gene, even non-HVG."*

#### Section 8 — Scaling + PCA
> *"Z-score scaling per gene, clipped at 10 standard deviations to stop
> outliers dominating. Then PCA — linear dimensionality reduction. The
> elbow plot tells me how many PCs carry signal — I'll feed ~40 into the
> neighbour graph."*

#### Section 9 — Neighbours + UMAP
> *"`sc.pp.neighbors` builds a KNN graph on the PCA embedding. UMAP is
> the 2D projection of that graph for visualisation only — important
> point: clustering runs on the **graph**, not on UMAP coordinates."*

#### Section 10 — Leiden clustering — **THIS IS YOUR JUDGMENT CALL**
> *"Leiden is community detection on the neighbour graph — same role as
> Seurat's `FindClusters`. Resolution controls granularity. There's no
> right answer; I run several and pick based on biological coherence."*

**Stop here.** Look at the three UMAPs. Pick one. Tell your partner why.

A good answer has the shape: *"I'd pick X because [biology] — at lower
resolution Y is lumped with Z which have clearly different markers; at
higher resolution the [population] fragments without obvious markers
separating the pieces."*

Even if your answer is *"I'd want to see the markers before deciding"* —
**that is also a valid interview answer.** Saying *"I don't have enough
information yet"* shows scientific maturity. Don't bluff.

#### Section 11 — Marker genes
> *"One-vs-rest Wilcoxon rank-sum per cluster. Same as `FindAllMarkers()`
> with default settings."*

#### Section 12 — Cell-type annotation
> *"Match clusters to canonical markers — EPCAM/KRT8 for epithelial,
> CD3D for T cells, CD14/LYZ for myeloid, MS4A1 for B cells. Manual
> annotation, not automated, because PBMC3k is small and unambiguous —
> for Smillie I'd consider an automated tool like CellTypist as a
> sanity check."*

#### Section 13 — Myeloid zoom-in (skipped on PBMC3k)
> *"On the real Smillie data this is where I sub-cluster the myeloid
> compartment to separate cDC1, cDC2, monocytes, macrophages — using
> markers like CLEC9A and XCR1 for cDC1, CLEC10A and CD1C for cDC2.
> This connects directly to my PhD work on DC progenitor biology."*

This is the section that maps onto the IHB team's interest. Have a
sentence ready about *why* you'd do it and *what you'd expect to see*.

---

## Phase 3 — Three rehearsed Q&As (7 min)

Read each question, write your answer in your own words (1–2 sentences),
then say it out loud once. The honest framing is **always** more
defensible than a polished answer that overclaims.

### Q1: "Tell me about your computational background."

**Honest framing:**
> *"My publication-level depth is in R with Seurat — I taught myself
> during my PhD and used it for real scRNA-seq analyses, including in
> [name a paper or DC project]. Python and Scanpy I've been ramping up
> on this month — I'm comfortable with the AnnData workflow and the
> conceptual pipeline, and I'd be confident moving fully into the
> Scverse stack on the job."*

**Avoid:** *"I'm experienced with Scanpy."* You aren't yet. The
interview is on day ~7 of you using it.

### Q2: "Walk me through your project."

**Structure:** dataset → why this dataset → what you did → judgment call → what's next.

**Honest framing:**
> *"I worked through the Smillie 2019 UC dataset because it's the canonical
> IBD scRNA-seq paper and covers the cell types your team studies. I went
> end-to-end in Scanpy: QC, doublet detection, normalisation, HVG, PCA,
> Leiden clustering, marker genes, and manual annotation, with a focus on
> the myeloid compartment because that's where my wet-lab background
> connects. I made a few judgment calls — for example I picked Leiden
> resolution X because [reason]. If I had more time, the next step is
> [Smillie download / sub-clustering / batch effects across donors]."*

### Q3: "Where did you make a judgment call you weren't sure about?"

This is the trick question — they want to see if you can **acknowledge
uncertainty**. Saying "no, every choice was obvious" is the wrong answer.

**Honest framing:**
> *"The Leiden resolution. There's no right answer — I picked X because
> the clusters mapped cleanly to canonical markers, but at higher
> resolution I would have separated [populations] which is biologically
> meaningful. In a real analysis I'd want to look at the dendrogram or
> stability metrics before committing — for this project I prioritised
> getting end-to-end first."*

Other valid "I wasn't sure" answers:
- The mito % cutoff (20% vs 10% — defensible but a real choice)
- Whether to regress out cell cycle (didn't here, would on Smillie)
- Whether to integrate across donors (didn't here, would on Smillie)

---

## Final checklist before the interview

- [ ] Can name the five AnnData slots without looking
- [ ] Can give the 30-second AnnData definition
- [ ] Can narrate at least 8 of 13 notebook sections without scrolling
- [ ] Have a Leiden resolution chosen with a one-sentence reason
- [ ] Have a one-sentence answer to all three rehearsed Qs
- [ ] Have re-read [NOTES.md](NOTES.md) once (R↔Python cheat sheet)
- [ ] Have one sentence on what you'd do *next* (Smillie download +
      myeloid sub-clustering)

If any of these are no, that's where the next 30 minutes goes. Don't
add new material.
