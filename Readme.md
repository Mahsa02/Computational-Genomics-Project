# Spatial Transcriptomics: Cellpose + Cosine Prototype Pipeline

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

Data files are located (here)[see link].

They are the following:
- blah blah
- blah blah x2

Data files are not tracked by git. See `.gitignore`.

## The pipeline

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
see requirements.txt and use python 3.13!

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
p