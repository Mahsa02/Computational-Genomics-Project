# Spatial Transcriptomics — Cellpose + Cosine Prototype Pipeline

Segments cells in Xenium spatial transcriptomics data using Cellpose, then assigns
each cell a biological identity by comparing its gene expression profile against
scRNA-seq-derived cell-type prototypes via cosine similarity.

## Overview

```
scRNA-seq data ──► 01_build_prototypes   ──► prototypes_normalized.csv
                                                      │
OME-TIFF image ──► 02_assign_transcripts ──► cell_expression_aligned.csv
transcripts.parquet                                   │
                                                      ▼
                   03_cosine_similarity  ──► cell_type_predictions.csv
```

## Repo structure

```
spatial-transcriptomics/
├── notebooks/
│   ├── 01_build_prototypes.ipynb       # scRNA-seq → prototype profiles
│   ├── 02_assign_transcripts.ipynb     # Cellpose segmentation + transcript assignment
│   └── 03_cosine_similarity.ipynb      # cosine similarity + cell-type labelling
├── data/                               # input and intermediate data (git-ignored)
├── outputs/                            # figures and QC plots (git-ignored)
└── README.md
```

## Data files required

Place the following in `data/` before running:

| File | Description |
|---|---|
| `morphology_focus_0000.ome.tif` | Full-resolution Xenium morphology image |
| `transcripts.parquet` | Xenium transcript table (x/y coordinates + gene names) |
| `scrna_expression.csv` | scRNA-seq count matrix (cells × genes) |
| `scrna_metadata.csv` | scRNA-seq cell metadata with a `cell_type` column |
| `xenium_gene_panel.txt` | List of genes in the Xenium panel (one per line) |

Data files are not tracked by git. See `.gitignore`.

## Running the pipeline

Run the notebooks **in order**:

1. `01_build_prototypes.ipynb` — produces `prototypes_normalized.csv` and `shared_genes.txt`
2. `02_assign_transcripts.ipynb` — produces `masks_center.npy` and `cell_expression_aligned.csv`
3. `03_cosine_similarity.ipynb` — produces `cell_type_predictions.csv` and all figures

Each notebook has a **Config** cell at the top where you can adjust paths and parameters.

## Key parameters

| Notebook | Parameter | Default | Notes |
|---|---|---|---|
| 02 | `CROP_SIZE` | `512` | Pixel size of centre crop |
| 02 | `CELL_DIAMETER` | `30` | Expected cell diameter for Cellpose |
| 02 | `GENE_COL` | `feature_name` | Gene column in `transcripts.parquet` — may be `gene_name` depending on Xenium version |
| 03 | `HEATMAP_N_CELLS` | `50` | Cells shown in similarity heatmap |

## Dependencies

```
cellpose
tifffile
pyarrow
pandas
numpy
scikit-learn
matplotlib
seaborn
```

Install with:
```bash
pip install cellpose tifffile pyarrow pandas numpy scikit-learn matplotlib seaborn
```

Cellpose GPU support requires a CUDA-compatible environment (e.g. Google Colab with GPU runtime).

## Background

**Cellpose** is a deep learning segmentation model that predicts a flow field pointing
each pixel toward its nearest cell centre, then traces boundaries from those flows.
This approach handles touching and overlapping cells well.

**Cosine similarity** measures the angle between two gene expression vectors regardless
of their magnitude. A score near 1 means a cell's expression profile points in the same
direction as a prototype — i.e. it strongly resembles that cell type.

The key design decision is that `shared_genes.txt` acts as the single source of truth
for gene ordering across all three notebooks, preventing silent misalignment between
the prototype matrix and the cell expression matrix.
