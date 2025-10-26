Author: Tiange Feng

Date: October, 2025

# Samples
Mouse gastric mucosa (NCD, Sham vs VSG)

# Project Summary
## 1. Seurat Standard Pre-processing and Dimensional Reduction

I began with a raw, unprocessed Seurat object containing 43 cell types. To prepare it for visualization, I wrote a complete pre-processing script:
* Data Preparation: NormalizeData, FindVariableFeatures, ScaleData.
* Dimensional Reduction: RunPCA and RunUMAP.
* Clustering: FindNeighbors and FindClusters (to create the seurat_clusters column).

## 2. Core Issue Troubleshooting

During this process, I encountered a non-finite entries error:

This was the biggest obstacle. I discovered it was caused by RunPCA attempting to calculate more principal components (e.g., 50 or 30) than the complexity of the data could support.

Solution: I solved this error by progressively lowering the npcs parameter in RunPCA (eventually to 20 or 25) and ensuring the dims parameter in RunUMAP matched it.

## 3. Data Filtering and Subsetting

I had several complex filtering needs, which I accomplished using different methods:

* Filtering by Cell Type (Metadata): I used two methods to highlight or filter for specific cell types (like "Enteroendocrine cells"):

        WhichCells(): Using the expression !!sym(column_name) == "value".

        FetchData(): First pulling the metadata, then manually filtering the rownames().

* Filtering by Gene Expression: I used GetAssayData and FetchData to retrieve expression data, and used logical tests like [expression[[marker]] > 0] to find marker-positive cells.

* Dual Filtering ("Metadata + Gene Expression"): This was an advanced requirement (e.g., filtering for cells that were a specific cell type and had Pparg > 0).

        Solution: My method was to first use GetAssayData to pull the gene's counts, add it as a new temporary column to sobj@meta.data, and then use the subset() function to filter on both metadata columns simultaneously.

## 4. Data Visualization

I progressed from basic plots to highly advanced, custom plots.

### DimPlot / FeaturePlot:

* Solved the multi-color assignment problem for 43 cell types (recommending hcl.colors()).
* Implemented faceted display by "treatment" (using split.by).
* Adjusted the plot's aspect ratio (coord.fixed, theme(aspect.ratio)).

### VlnPlot (Basic):

* Learned how to adjust the y-axis range (+ ylim(0, 5)).
* Resolved the None of the cells requested found error caused by the idents parameter (fixed by either subset()-ing the object first or setting Idents(sobj) <- ... beforehand).

### VlnPlot (Advanced - Manual ggplot2, patchwork Function):

* A requirement that VlnPlot(stack=TRUE) could not handle: filtering out zero-expression cells individually for each of the 5 stacked genes.

  Solution: I wrote an advanced script using tidyr::pivot_longer and dplyr::left_join to merge "normalized expression data," "raw count data," and "cell type metadata" into a single "long-format data frame," then manually plotted the filtered data using ggplot2 and facet_wrap(ncol=1).

* Found a StackedVlnPlot function written using patchwork and purrr::map. Made modifications to enhance its functionality:

      Added a group.by parameter to specify the x-axis.

      Implemented a cols parameter for custom fill colors.

      Modified the modify_vlnplot helper function to make the violin outline color (color) automatically match the fill color (fill).

      Changed the y-axis ticks from "show max value only" to "show both 0 and the max value".
