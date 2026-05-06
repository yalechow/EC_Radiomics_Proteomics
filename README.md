# EC-Radiomics-Proteomics-LNM

## Overview

This repository contains the data for the study on **predicting lymph node metastasis (LNM) in endometrial carcinoma (EC)** using a multimodal radiomics-proteomics approach.

The study developed a 20-feature radiomics signature from multi-sequence MRI (CE + DWI + T2) using a Logistic Regression model (L2, class-weight balanced) trained on 432 cases. Radiomics scores were subsequently correlated with quantitative proteomics data from 32 FFPE samples to identify hub proteins bridging imaging phenotypes and molecular biology. Additionally, transcriptomic differentially expressed genes (DEGs) from GEO cohort GSE120490 are provided to contextualize the LNM-associated molecular landscape.

**Internal validation (repeated stratified 5-fold CV × 3 repeats):**
- AUC: 0.840
- Sensitivity: 0.797 | Specificity: 0.736

---

## Repository Structure

```
EC_Radiomics_Proteomics_GitHub/
├── README.md
├── data/
│   ├── radiomics_features_20.csv         # 20 selected radiomics features + RadScore (32 matched samples)
│   ├── proteomics_32samples.csv          # Top-10 hub protein intensities from FFPE proteomics (32 samples)
│   ├── hub_proteins_correlation.csv      # Pearson correlation: RadScore vs hub proteins (|r| ≥ 0.4)
│   ├── proteomics/
│   │   ├── protein_quantitation.xlsx     # Full protein quantitation (all identified proteins, 32 tumor samples)
│   │   ├── protein_expression_matrix.xlsx# Normalized expression matrix for differential analysis
│   │   ├── DEP_LNMpos_vs_LNMneg.xlsx    # Differentially expressed proteins: LNM+ vs LNM−
│   │   └── QC_plot_data.xlsx             # QC summary data (RSD, PCA scores, sample correlation)
│   └── transcriptomics/
│       ├── GSE120490_clinical.csv        # Sample clinical metadata (LNM status, etc.)
│       ├── GPL570_probe_annotation.csv   # Affymetrix GPL570 probe-to-gene mapping
│       ├── GSE120490_expression_matrix.csv # RMA-normalized expression matrix
│       ├── GSE120490_limma_full.csv      # Full limma output for all probes
│       ├── GSE120490_degs_sig.csv        # 379 significant DEGs (adj.P.Val < 0.05)
│       ├── GSE120490_degs_up.csv         # 215 upregulated DEGs
│       └── GSE120490_degs_down.csv       # 164 downregulated DEGs
├── results/
│   ├── internal_validation/
│   │   ├── model_performance.csv         # Apparent + cross-validation metrics
│   │   ├── model_benchmark_summary.csv   # All selector × classifier combinations (165 total, 1 selected)
│   │   ├── repeated_cv_split_metrics.csv # Per-fold CV metrics
│   │   └── repeated_cv_calibration_curve.csv
│   └── feature_selection/
│       ├── selected_features.txt         # Final 20 features
│       ├── model_coefficients.csv        # 20 features + coefficients + scaler params (model input order)
│       ├── reduction_trace.csv           # ICC → PCC → RFE step trace
│       ├── pcc_decisions.csv             # PCC redundancy filter decisions
│       └── rfe_selector_ranking.csv      # RFE feature ranking
└── figures/
    ├── GSE120490_volcano.pdf
    ├── mri_examples/                     # Representative CE / DWI / T2 slices
    ├── proteomics_QC/                    # PCA, correlation, RSD, identified proteins
    ├── proteomics_DEP/                   # Volcano + heatmap (LNM+ vs LNM−)
    ├── proteomics_enrichment/            # GO / KEGG bar & bubble plots
    └── proteomics_PPI/                   # PPI network plots
```

---

## Data Description

### Radiomics Features (`data/radiomics_features_20.csv`)

- **Samples**: 32 cases with matched radiomics and proteomics data (anonymized as `Sample_001`–`Sample_032`)
- **Columns**: `Sample_ID`, `LNM` (lymph node metastasis label: 1=positive, 0=negative), 20 radiomics features, `RadScore`, `Predicted_Probability`
- **Feature pipeline**: ICC ≥ 0.75 → PCC redundancy filter (threshold 0.8) → RFE (k=20) → Logistic Regression (L2)
- **Sequences**: CE (contrast-enhanced), DWI (diffusion-weighted), T2

### Proteomics (`data/proteomics_32samples.csv`)

- **Samples**: 32 FFPE tumor tissue samples (same cohort as radiomics, anonymized)
- **Method**: Tandem Mass Spectrometry quantitative proteomics
- **Columns**: `Sample_ID`, top-10 hub proteins (ESR1, RSPH1, GRIA2, IGF1, TPPP3, CAPSL, SPATA18, ARMC3, ERMN, GPM6B)
- **Values**: Log2-transformed protein intensities

### Proteomics Results (`data/proteomics/`)

Derived from FFPE quantitative proteomics (tandem mass spectrometry) on 32 endometrial carcinoma samples (16 LNM+, 16 LNM−).

| File | Description |
|------|-------------|
| `protein_quantitation.xlsx` | Full protein quantitation table (all identified proteins) |
| `protein_expression_matrix.xlsx` | Normalized expression matrix used for differential analysis |
| `DEP_LNMpos_vs_LNMneg.xlsx` | Differentially expressed proteins: LNM+ vs LNM− (key comparison) |
| `QC_plot_data.xlsx` | QC summary data (RSD, PCA scores, sample correlation) |

### Hub Protein Correlation (`data/hub_proteins_correlation.csv`)

- Pearson correlation between RadScore and each hub protein
- Filtered for |r| ≥ 0.4, FDR-adjusted p-value
- Includes protein annotation (regulation direction, log2FoldChange from proteomics discovery)

### GSE120490 Transcriptomics (`data/transcriptomics/`)

- **Source**: GEO dataset GSE120490 (Endometrial carcinoma, LNM-related; GPL570 Affymetrix Human Genome U133 Plus 2.0 Array)
- **Raw data**: Available at [NCBI GEO GSE120490](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE120490)

| File | Description | Size |
|------|-------------|------|
| `GSE120490_clinical.csv` | Sample clinical metadata (LNM status, etc.) | 8 KB |
| `GPL570_probe_annotation.csv` | Affymetrix GPL570 probe-to-gene mapping | 1.2 MB |
| `GSE120490_expression_matrix.csv` | RMA-normalized expression matrix (all samples) | 23 MB |
| `GSE120490_limma_full.csv` | Full limma output for all probes (logFC, t, P.Value, adj.P.Val) | 2.3 MB |
| `GSE120490_degs_sig.csv` | **379 significant DEGs** (adj.P.Val < 0.05) | 44 KB |
| `GSE120490_degs_up.csv` | 215 upregulated DEGs | 25 KB |
| `GSE120490_degs_down.csv` | 164 downregulated DEGs | 19 KB |

- Analysis method: limma differential expression (LNM+ vs LNM−)
- Columns in DEG files: `gene_symbol`, `logFC`, `AveExpr`, `t`, `P.Value`, `adj.P.Val`, `B`

---

## Results

### Internal Validation (`results/internal_validation/`)

| File | Description |
|------|-------------|
| `model_performance.csv` | Apparent and cross-validation metrics (AUC, sensitivity, specificity, calibration, etc.) |
| `model_benchmark_summary.csv` | All selector × classifier combinations with CV metrics |
| `repeated_cv_split_metrics.csv` | Per-fold metrics across all CV repeats |
| `repeated_cv_calibration_curve.csv` | Calibration curve data (mean predicted vs. observed rate) |

### Feature Selection Trace (`results/feature_selection/`)

| File | Description |
|------|-------------|
| `selected_features.txt` | Final 20 selected features |
| `model_coefficients.csv` | Final 20 features with logistic regression coefficients + scaler parameters (ordered by model input) |
| `reduction_trace.csv` | Step-by-step feature count: ICC → PCC → RFE |
| `pcc_decisions.csv` | PCC redundancy filter: kept/dropped feature pairs |
| `rfe_selector_ranking.csv` | RFE ranking for all candidate features |

---

## Figures

### Transcriptomics (`figures/`)
| File | Description |
|------|-------------|
| `GSE120490_volcano.pdf` | DEG volcano plot (GSE120490, limma) |

### MRI Representative Images (`figures/mri_examples/`)
Representative MRI slices (CE, DWI, T2) from a single case illustrating the segmented tumor region.

### Proteomics QC (`figures/proteomics_QC/`)
| File | Description |
|------|-------------|
| `PCA_all_samples.pdf/png` | PCA plot of all 32 proteomics samples |
| `sample_correlation.pdf/png` | Inter-sample Pearson correlation heatmap |
| `RSD_boxplot.pdf/png` | RSD distribution boxplot (technical reproducibility) |
| `identified_proteins_per_sample.pdf/png` | Number of proteins identified per sample |

### Differentially Expressed Proteins (`figures/proteomics_DEP/`)
Key comparison: **LNM+ tumor vs LNM− tumor** (Experimental_cancer vs Control_cancer)

| File | Description |
|------|-------------|
| `volcano_LNMpos_vs_LNMneg.pdf/png` | Volcano plot of DEPs |
| `heatmap_LNMpos_vs_LNMneg.pdf/png` | Hierarchical clustering heatmap of DEPs |

### Functional Enrichment (`figures/proteomics_enrichment/`)
GO and KEGG enrichment for DEPs (LNM+ vs LNM−):
`GO_bubble_total`, `GO_bar_total`, `KEGG_bubble_total`, `KEGG_bar_total` (PDF + PNG)

### PPI Network (`figures/proteomics_PPI/`)
Protein–protein interaction network of DEPs (LNM+ vs LNM−):
`PPI_network_gene.pdf/png`, `PPI_network_query.pdf/png`

---

## Data Availability

- **Raw MRI images and clinical data**: Not publicly available due to patient privacy regulations.
- **GSE120490**: Available at [NCBI GEO](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE120490)

---

## Citation

> *[Manuscript under review – citation will be added upon publication]*

---

## License

This repository is released under the [MIT License](LICENSE).
