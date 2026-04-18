# Biologically-Informed Cell Segmentation via Mesmer Loss Modification

This project explores whether cell segmentation quality can be improved by incorporating biological signal — specifically, single-cell RNA sequencing (scRNA-seq) reference data — into the training objective of [Mesmer](https://www.nature.com/articles/s41587-021-01094-0), a deep learning model for multiplexed tissue image segmentation.

The core idea: standard Mesmer is trained purely on image features (nuclear and membrane markers). We hypothesize that a segmentation is "better" if the gene expression profile reconstructed from each predicted cell closely resembles a known cell type. We operationalize this using **cosine similarity** between per-cell expression vectors and scRNA-seq prototypes, with the long-term goal of incorporating this signal as an auxiliary loss term during fine-tuning.


## Project Structure
```
.
├── README.md
├── build_prototypes.ipynb     # Part 1 — scRNA-seq prototype construction
├── segmentation_pipeline.ipynb  # Part 2 — Mesmer segmentation + expression matrix
├── cosine_similarity.ipynb    # Part 3 — prototype matching + evaluation
└── data/
    ├── shared_genes.txt         # master gene list (locked ordering)
    ├── prototypes_raw.csv       # mean expression per cell type (unnormalized)
    ├── prototypes_norm.csv      # L2-normalized prototypes (K × G)
    ├── prototypes_norm.npy
    ├── adata_sp_processed.h5ad  # processed spatial data
    ├── adata_sc_processed.h5ad  # processed scRNA-seq data
    ├── segmentation_mask.npy    # Mesmer output — per-pixel cell IDs
    ├── cell_expression.npy      # reconstructed cell × gene matrix (C × G)
└── figures/
    ├── segmentation_overview.png
    └── spot_assignment.png
```

## What Each part/notebook is about

### 1 — scRNA-seq Prototypes (`build_prototypes.ipynb`)
Loads a matched scRNA-seq reference dataset and a 10x Visium spatial transcriptomics dataset, harmonizes gene names, subsets both to shared genes, and computes a **prototype vector** for each cell type (mean log-normalized expression across all cells of that type). These prototypes serve as the biological ground truth that segmented cells will later be compared against.

**Output:** `prototypes_norm.npy` (shape: K × G), `shared_genes.txt`

### 2 — Segmentation + Expression Matrix (`segmentation_pipeline.ipynb`)
Loads multiplexed tissue images and runs Mesmer to produce a cell segmentation mask. Then assigns each spatial transcript (spot) to its corresponding segmented cell and aggregates expression to build a **cell × gene matrix**. Gene ordering is locked to Person 1's `shared_genes.txt`.

**Output:** `cell_expression.npy` (shape: C × G), `segmentation_mask.npy`

### 3 — Cosine Similarity + Evaluation (`cosine_similarity.ipynb`)
Loads the prototype matrix (Person 1) and cell expression matrix (Person 2), computes cosine similarity between every cell and every prototype, assigns predicted cell types, and produces evaluation figures. This establishes whether biological signal is detectable in Mesmer's output — a prerequisite for incorporating it as a loss term.

**Output:** similarity scores, figures, interpretation notes

## How This Connects to the Loss Function

The pipeline as described is **Phase 1: evaluation**. The goal is to confirm that cosine similarity between segmented cell expression and scRNA-seq prototypes produces a meaningful signal before attempting to modify Mesmer's training.

If Phase 1 succeeds, Phase 2 involves incorporating this signal into Mesmer's loss during fine-tuning:

`L_total = L_segmentation + λ · L_cosine`

where `L_cosine` penalizes segmentations whose reconstructed per-cell expression vectors are dissimilar to all known prototypes. The key technical challenge is that the transcript-to-cell assignment step is non-differentiable as written — making this differentiable (e.g., via soft assignment) is the core open problem.

## Data

The datasets used are not included in this repository. To obtain them, see [drive](https://drive.google.com/drive/folders/1zu7sYOevH4cgKzgydl4YucbjF4j17fMr?usp=share_link).

- **Spatial:** `CytAssist_Fresh_Frozen_Mouse_Brain_filtered_feature_bc_matrix.h5` — 10x Visium mouse brain
- **scRNA-seq:** `90cdd3f7-e61e-43ed-97e9-ddc0f7e87827.h5ad` — matched single-cell reference

Once downloaded, update `DATA_DIR` at the top of each notebook to point to the folder containing these files.


## Integration Checklist

Before running Part 3, confirm:

- [ ] `shared_genes.txt` was produced
- [ ] `prototypes_norm.npy` shape is **(K × G)**
- [ ] `cell_expression.npy` shape is **(C × G)**
- [ ] G (number of genes) matches across both — should equal `len(shared_genes.txt)`
- [ ] Both matrices used the same normalization scheme (`normalize_total` + `log1p`)

## Acknowledgements

Advised by Prof. Cassandra Burdziak. [Mesmer](https://pubmed.ncbi.nlm.nih.gov/34795433/) was developed by Greenwald et al. (2022) at the Angelo Lab; [the DeepCell framework][-(https://github.com/vanvalenlab/deepcell-tf) is maintained by the Van Valen Lab at Caltech.
