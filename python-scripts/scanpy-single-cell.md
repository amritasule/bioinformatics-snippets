# Scanpy Single-Cell RNA-seq Analysis

## Setup & Import
```python
import scanpy as sc
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

# Set up scanpy settings
sc.settings.verbosity = 3  # errors (0), warnings (1), info (2), hints (3)
sc.settings.set_figure_params(dpi=80, facecolor='white')
sc.logging.print_versions()
```

---

## Data Loading

### Load h5ad file
```python
adata = sc.read_h5ad('data.h5ad')
```

### Load 10x data
```python
adata = sc.read_10x_mtx(
    'path/to/cellranger/outs/filtered_feature_bc_matrix/',
    var_names='gene_symbols',
    cache=True
)
```

### Load from CSV/TSV
```python
adata = sc.read_csv('counts.csv').T  # Transpose if genes are rows
```

### Quick inspection
```python
adata  # Print basic info
adata.obs.head()  # Cell metadata
adata.var.head()  # Gene metadata
```

---

## Quality Control

### Calculate QC metrics
```python
# Calculate mitochondrial/ribosomal genes
adata.var['mt'] = adata.var_names.str.startswith('MT-')
adata.var['ribo'] = adata.var_names.str.startswith(('RPS', 'RPL'))

# Calculate QC metrics
sc.pp.calculate_qc_metrics(
    adata, 
    qc_vars=['mt', 'ribo'], 
    percent_top=None, 
    log1p=False, 
    inplace=True
)
```

### Visualize QC
```python
# Violin plots
sc.pl.violin(adata, ['n_genes_by_counts', 'total_counts', 'pct_counts_mt'],
             jitter=0.4, multi_panel=True)

# Scatter plots
sc.pl.scatter(adata, x='total_counts', y='pct_counts_mt')
sc.pl.scatter(adata, x='total_counts', y='n_genes_by_counts')
```

### Filter cells and genes
```python
# Filter cells
sc.pp.filter_cells(adata, min_genes=200)
sc.pp.filter_cells(adata, max_genes=5000)

# Filter by mitochondrial content
adata = adata[adata.obs.pct_counts_mt < 20, :]

# Filter genes
sc.pp.filter_genes(adata, min_cells=3)
```

---

## Preprocessing

### Normalize and log-transform
```python
# Save raw counts
adata.raw = adata

# Normalize to 10,000 reads per cell
sc.pp.normalize_total(adata, target_sum=1e4)

# Log-transform
sc.pp.log1p(adata)
```

### Highly variable genes
```python
sc.pp.highly_variable_genes(adata, n_top_genes=2000)

# Plot
sc.pl.highly_variable_genes(adata)

# Keep only HVGs (optional)
# adata = adata[:, adata.var.highly_variable]
```

### Scale and regress
```python
# Regress out unwanted sources of variation
sc.pp.regress_out(adata, ['total_counts', 'pct_counts_mt'])

# Scale to unit variance and zero mean
sc.pp.scale(adata, max_value=10)
```

---

## Dimensionality Reduction

### PCA
```python
sc.tl.pca(adata, svd_solver='arpack', n_comps=50)

# Visualize
sc.pl.pca_variance_ratio(adata, log=True, n_pcs=50)
sc.pl.pca(adata, color='sample_id')
```

### Batch Correction with Harmony
```python
import scanpy.external as sce

# Run Harmony
sce.pp.harmony_integrate(adata, key='batch_key', max_iter_harmony=20)

# Use harmony-corrected PCA for downstream
adata.obsm['X_pca'] = adata.obsm['X_pca_harmony']
```

### Neighborhood graph
```python
sc.pp.neighbors(adata, n_neighbors=10, n_pcs=30)
```

### UMAP
```python
sc.tl.umap(adata, min_dist=0.3)

# Plot
sc.pl.umap(adata, color=['sample_id', 'n_genes'])
```

---

## Clustering

### Leiden clustering
```python
# Try different resolutions
sc.tl.leiden(adata, resolution=0.5, key_added='leiden_0.5')
sc.tl.leiden(adata, resolution=1.0, key_added='leiden_1.0')

# Visualize
sc.pl.umap(adata, color=['leiden_0.5', 'leiden_1.0'])
```

### Louvain clustering (alternative)
```python
sc.tl.louvain(adata, resolution=0.5)
```

---

## Differential Expression & Marker Genes

### Find marker genes per cluster
```python
sc.tl.rank_genes_groups(adata, 'leiden', method='wilcoxon')

# Visualize top markers
sc.pl.rank_genes_groups(adata, n_genes=25, sharey=False)

# Get results as dataframe
result = sc.get.rank_genes_groups_df(adata, group=None)
```

### Visualize specific genes
```python
# Dotplot
sc.pl.dotplot(adata, marker_genes_dict, groupby='leiden')

# Heatmap
sc.pl.heatmap(adata, marker_genes_dict, groupby='leiden')

# UMAP
sc.pl.umap(adata, color=['CD3D', 'CD8A', 'CD4'], use_raw=True)

# Violin plot
sc.pl.violin(adata, ['CD3D', 'CD8A'], groupby='leiden')

# Stacked violin
sc.pl.stacked_violin(adata, marker_genes_dict, groupby='leiden')
```

---

## Cell Type Annotation

### Manual annotation based on markers
```python
# Define cluster to cell type mapping
cluster_to_celltype = {
    '0': 'CD4+ T cells',
    '1': 'CD8+ T cells',
    '2': 'NK cells',
    '3': 'B cells',
    '4': 'Monocytes',
    '5': 'Dendritic cells'
}

adata.obs['cell_type'] = adata.obs['leiden'].map(cluster_to_celltype)

# Visualize
sc.pl.umap(adata, color='cell_type')
```

### Score gene sets
```python
# Define marker genes for cell types
cd4_markers = ['CD3D', 'CD4', 'IL7R']
cd8_markers = ['CD3D', 'CD8A', 'CD8B']

# Calculate scores
sc.tl.score_genes(adata, cd4_markers, score_name='CD4_score')
sc.tl.score_genes(adata, cd8_markers, score_name='CD8_score')

# Visualize
sc.pl.umap(adata, color=['CD4_score', 'CD8_score'])
```

---

## Compositional Analysis

### Cell type proportions
```python
# Calculate proportions per sample
cell_counts = pd.crosstab(adata.obs['sample_id'], adata.obs['cell_type'])
cell_props = cell_counts.div(cell_counts.sum(axis=1), axis=0)

# Plot
cell_props.plot(kind='bar', stacked=True, figsize=(12, 6))
plt.legend(bbox_to_anchor=(1.05, 1))
plt.ylabel('Proportion')
plt.tight_layout()
```

### scCODA for compositional analysis
```python
# Install: pip install sccoda
from sccoda.util import comp_ana as mod
from sccoda.util import cell_composition_data as dat

# Prepare data
sccoda_data = dat.from_scanpy(
    adata, 
    cell_type_identifier='cell_type',
    sample_identifier='sample_id'
)

# Run model
sccoda_model = mod.CompositionalAnalysis(sccoda_data, formula="condition")
sccoda_results = sccoda_model.sample_hmc()

# Get credible effects
sccoda_results.summary()
```

---

## Subsetting & Analysis

### Subset by cell type
```python
# Keep only T cells
tcells = adata[adata.obs['cell_type'].isin(['CD4+ T cells', 'CD8+ T cells'])].copy()

# Re-analyze subset
sc.pp.neighbors(tcells, n_pcs=30)
sc.tl.umap(tcells)
sc.tl.leiden(tcells, resolution=0.5)
```

### Subset by sample/condition
```python
# Keep specific samples
subset = adata[adata.obs['sample_id'].isin(['sample1', 'sample2'])].copy()
```

---

## Saving Results

### Save annotated data
```python
adata.write('results/annotated_data.h5ad')
```

### Export to CSV
```python
# Cell metadata
adata.obs.to_csv('results/cell_metadata.csv')

# UMAP coordinates
pd.DataFrame(
    adata.obsm['X_umap'],
    columns=['UMAP1', 'UMAP2'],
    index=adata.obs_names
).to_csv('results/umap_coords.csv')

# Marker genes
markers = sc.get.rank_genes_groups_df(adata, group=None)
markers.to_csv('results/marker_genes.csv', index=False)
```

---

## Useful Tips

### Memory management
```python
# Delete unused data
del adata.raw
del adata.uns['log1p']

# Force garbage collection
import gc
gc.collect()
```

### Working with large datasets
```python
# Use backed mode (doesn't load entire matrix into memory)
adata = sc.read_h5ad('large_data.h5ad', backed='r')

# Process in chunks
for chunk in adata.obs['sample_id'].unique():
    subset = adata[adata.obs['sample_id'] == chunk].to_memory()
    # Process subset
```

### Copy vs. view
```python
# Always copy when subsetting for further analysis
subset = adata[mask].copy()  # Creates independent copy

# View is just a reference (modifying changes original)
view = adata[mask]  # Don't use for independent analysis
```

---

## Standard Analysis Pipeline
```python
# 1. Load and QC
adata = sc.read_h5ad('data.h5ad')
sc.pp.calculate_qc_metrics(adata, qc_vars=['mt'], inplace=True)
adata = adata[adata.obs.pct_counts_mt < 20]

# 2. Normalize and identify HVGs
adata.raw = adata
sc.pp.normalize_total(adata, target_sum=1e4)
sc.pp.log1p(adata)
sc.pp.highly_variable_genes(adata, n_top_genes=2000)

# 3. Scale and PCA
sc.pp.regress_out(adata, ['total_counts', 'pct_counts_mt'])
sc.pp.scale(adata, max_value=10)
sc.tl.pca(adata)

# 4. Batch correction (if needed)
sce.pp.harmony_integrate(adata, key='batch')

# 5. Clustering
sc.pp.neighbors(adata, n_pcs=30)
sc.tl.umap(adata)
sc.tl.leiden(adata, resolution=0.5)

# 6. Find markers and annotate
sc.tl.rank_genes_groups(adata, 'leiden')
# Manual annotation...

# 7. Save
adata.write('results/processed.h5ad')
```

---

**Created:** November 26, 2024
**Last Updated:** November 26, 2024