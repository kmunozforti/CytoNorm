Use case 3: Normalize channels without including them in the clustering
step
================

<!-- github markdown built using
rmarkdown::render("vignettes/CytoNorm_channels_without_clustering.Rmd", output_format = "github_document")
-->

#### In this vignette, you learn how CytoNorm can be used to normalize markers that are not used during the clustering step.

One of the strengths of CytoNorm and some other normalization algorithms
is the clustering step. By clustering first, cell subsets are separated,
which makes itis possible to apply different normalizations on the
different subsets. However,in cytometry, we often make the distinction
between phenotypic or cell type markers and functional or cell state
markers and clustering is typically done only with the phenotypic
markers, as functional markers typically do not contribute to the
clustering, or even reduce the clustering quality. Therefore, it is
important that the normalization algorithm allows making a
distinctionbetween the channels that need to be normalized, which might
include functional markers, and the channels that will be used for the
clustering.

By default, CytoNorm uses all the markers specified for normalization
also for clustering. It is however straightforward to specify different
sets of markers for the clustering step (in the FlowSOM parameters) and
for the actualnormalization procedure.

## Prepare the CytoNorm normalization step

The phenotypic markers of our mesothelioma dataset were already
normalized with the two models in the previous use cases ([Use case 1:
Run CytoNorm on a dataset without dedicated controls
vignette](CytoNorm_without_controls.md) and [Use case 2: Run CytoNorm to
normalize towards a distribution
vignette](CytoNorm_towards_distribution.md). Since we want to further
characterize the immune landscape, the next step was is analyze the
functional markers. A manual gating strategy of these markers was
designed and optimized by an expert for the first cohort of data (P1_C1
and P2_C1), which should be translated to the second cohort (P1_C2 and
P2_C2).

To do this, we train a CytoNorm model per panel where we set the goal to
the first cohort (P1_C1 or P2_C1), use the cell type markers for the
FlowSOMclustering and specify which (panel-specific) cell state markers
should be normalized.

``` r
library(CytoNorm)
library(flowCore)
library(FlowSOM)
library(ggpubr)
```

## Train the CytoNorm model

### List the files per batch that will be aggregated

``` r
dir_prepr <- "Data/Preprocessed"

# Organize preprocessed data in batches
files_P1_C2 <- list.files(path = "Data/Preprocessed/",
                          pattern = "ID[5-8]_Panel1_TP[1-3].fcs",
                          full.names = TRUE)

# List files (already normalized for cell type markers in use cases 1 and 2)
files_P1_C1_norm <- list.files(path = "Data/Normalized_cellType", 
                               pattern = "ID[1-4]_Panel1_TP..fcs", full.names = TRUE)
files_P1_C2_norm <- list.files(path = "Data/Normalized_cellType", 
                               pattern = "ID[5-8]_Panel1_TP..fcs", full.names = TRUE)

# Read in 1 file per panel to get channel information
ff_P1 <- read.FCS(files_P1_C1_norm[1])

# Retrieve channel information
cellTypeMarkers <- c("CD27", "CD38", "IgD", "IgM") 
cellTypeChannels <- GetChannels(object = ff_P1, markers = cellTypeMarkers)
P1_cellStateMarkers <- c("CD5", "CD268", "CD24", "PDL1", "PD1", "CD86", "KI67")
P1_cellStateChannels <- GetChannels(object = ff_P1, markers = P1_cellStateMarkers)

# Read in manualLabels object that was generated in the CytoNorm_without_controls vignette
manualLabels <- readRDS("Data/Preprocessed/attachments/manualLabels.rds")
```

### Train the CytoNorm model

``` r
model3_cellStateMarkersP1 <- CytoNorm.train(files = c(files_P1_C1_norm, files_P1_C2_norm),
                                            labels = c(rep(x = "C1",
                                                           times = length(files_P1_C1_norm)),
                                                       rep(x = "C2",
                                                           times = length(files_P1_C2_norm))),
                                            channels = P1_cellStateChannels,
                                            transformList = NULL,
                                            seed = 1,
                                            plot = TRUE,
                                            verbose = TRUE,
                                            normParams = list("goal" = "C1"),
                                            FlowSOM.params = list(nCells = 1e+06, 
                                                                  xdim = 7, ydim = 7, 
                                                                  nClus = 3, 
                                                                  scale = FALSE,
                                                                  colsToUse = cellTypeChannels))
#> Warning in CytoNorm.train(files = c(files_P1_C1_norm, files_P1_C2_norm), : Reusing FlowSOM result previously saved at ./tmp/CytoNorm_FlowSOM.RDS. 
#>          If this was not intended, one can either specify another outputDir, 
#>          make use of the recompute parameter or move the FlowSOM object in the 
#>          file manager.
#> Splitting Data/Normalized_cellType/ID1_Panel1_TP1.fcs
#> Splitting Data/Normalized_cellType/ID1_Panel1_TP2.fcs
#> Splitting Data/Normalized_cellType/ID1_Panel1_TP3.fcs
#> Splitting Data/Normalized_cellType/ID2_Panel1_TP1.fcs
#> Splitting Data/Normalized_cellType/ID2_Panel1_TP2.fcs
#> Splitting Data/Normalized_cellType/ID2_Panel1_TP3.fcs
#> Splitting Data/Normalized_cellType/ID3_Panel1_TP1.fcs
#> Splitting Data/Normalized_cellType/ID3_Panel1_TP2.fcs
#> Splitting Data/Normalized_cellType/ID3_Panel1_TP3.fcs
#> Splitting Data/Normalized_cellType/ID4_Panel1_TP1.fcs
#> Splitting Data/Normalized_cellType/ID4_Panel1_TP2.fcs
#> Splitting Data/Normalized_cellType/ID4_Panel1_TP3.fcs
#> Splitting Data/Normalized_cellType/ID5_Panel1_TP1.fcs
#> Splitting Data/Normalized_cellType/ID5_Panel1_TP2.fcs
#> Splitting Data/Normalized_cellType/ID5_Panel1_TP3.fcs
#> Splitting Data/Normalized_cellType/ID6_Panel1_TP1.fcs
#> Splitting Data/Normalized_cellType/ID6_Panel1_TP2.fcs
#> Splitting Data/Normalized_cellType/ID6_Panel1_TP3.fcs
#> Splitting Data/Normalized_cellType/ID7_Panel1_TP1.fcs
#> Splitting Data/Normalized_cellType/ID7_Panel1_TP2.fcs
#> Splitting Data/Normalized_cellType/ID7_Panel1_TP3.fcs
#> Splitting Data/Normalized_cellType/ID8_Panel1_TP1.fcs
#> Splitting Data/Normalized_cellType/ID8_Panel1_TP2.fcs
#> Splitting Data/Normalized_cellType/ID8_Panel1_TP3.fcs
#> Warning in FlowSOM::NewData(fsom, ff): 566 cells (0.83%) seem far from their cluster centers.
#> Processing cluster 1
#> Computing Quantiles
#>   C1 (FileID 1,2,3,4,5,6,7,8,9,10,11,12)
#> Reading ./tmp/Data_Normalized_cellType_ID1_Panel1_TP1.fcs_fsom1.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID1_Panel1_TP2.fcs_fsom1.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID1_Panel1_TP3.fcs_fsom1.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID2_Panel1_TP1.fcs_fsom1.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID2_Panel1_TP2.fcs_fsom1.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID2_Panel1_TP3.fcs_fsom1.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID3_Panel1_TP1.fcs_fsom1.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID3_Panel1_TP2.fcs_fsom1.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID3_Panel1_TP3.fcs_fsom1.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID4_Panel1_TP1.fcs_fsom1.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID4_Panel1_TP2.fcs_fsom1.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID4_Panel1_TP3.fcs_fsom1.fcs
#>   C2 (FileID 13,14,15,16,17,18,19,20,21,22,23,24)
#> Reading ./tmp/Data_Normalized_cellType_ID5_Panel1_TP1.fcs_fsom1.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID5_Panel1_TP2.fcs_fsom1.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID5_Panel1_TP3.fcs_fsom1.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID6_Panel1_TP1.fcs_fsom1.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID6_Panel1_TP2.fcs_fsom1.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID6_Panel1_TP3.fcs_fsom1.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID7_Panel1_TP1.fcs_fsom1.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID7_Panel1_TP2.fcs_fsom1.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID7_Panel1_TP3.fcs_fsom1.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID8_Panel1_TP1.fcs_fsom1.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID8_Panel1_TP2.fcs_fsom1.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID8_Panel1_TP3.fcs_fsom1.fcs
#> Computing Splines
#>   C1
#>   C2
#> Processing cluster 2
#> Computing Quantiles
#>   C1 (FileID 1,2,3,4,5,6,7,8,9,10,11,12)
#> Reading ./tmp/Data_Normalized_cellType_ID1_Panel1_TP1.fcs_fsom2.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID1_Panel1_TP2.fcs_fsom2.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID1_Panel1_TP3.fcs_fsom2.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID2_Panel1_TP1.fcs_fsom2.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID2_Panel1_TP2.fcs_fsom2.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID2_Panel1_TP3.fcs_fsom2.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID3_Panel1_TP1.fcs_fsom2.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID3_Panel1_TP2.fcs_fsom2.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID3_Panel1_TP3.fcs_fsom2.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID4_Panel1_TP1.fcs_fsom2.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID4_Panel1_TP2.fcs_fsom2.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID4_Panel1_TP3.fcs_fsom2.fcs
#>   C2 (FileID 13,14,15,16,17,18,19,20,21,22,23,24)
#> Reading ./tmp/Data_Normalized_cellType_ID5_Panel1_TP1.fcs_fsom2.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID5_Panel1_TP2.fcs_fsom2.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID5_Panel1_TP3.fcs_fsom2.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID6_Panel1_TP1.fcs_fsom2.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID6_Panel1_TP2.fcs_fsom2.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID6_Panel1_TP3.fcs_fsom2.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID7_Panel1_TP1.fcs_fsom2.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID7_Panel1_TP2.fcs_fsom2.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID7_Panel1_TP3.fcs_fsom2.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID8_Panel1_TP1.fcs_fsom2.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID8_Panel1_TP2.fcs_fsom2.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID8_Panel1_TP3.fcs_fsom2.fcs
#> Computing Splines
#>   C1
#>   C2
#> Processing cluster 3
#> Computing Quantiles
#>   C1 (FileID 1,2,3,4,5,6,7,8,9,10,11,12)
#> Reading ./tmp/Data_Normalized_cellType_ID1_Panel1_TP1.fcs_fsom3.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID1_Panel1_TP2.fcs_fsom3.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID1_Panel1_TP3.fcs_fsom3.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID2_Panel1_TP1.fcs_fsom3.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID2_Panel1_TP2.fcs_fsom3.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID2_Panel1_TP3.fcs_fsom3.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID3_Panel1_TP1.fcs_fsom3.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID3_Panel1_TP2.fcs_fsom3.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID3_Panel1_TP3.fcs_fsom3.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID4_Panel1_TP1.fcs_fsom3.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID4_Panel1_TP2.fcs_fsom3.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID4_Panel1_TP3.fcs_fsom3.fcs
#>   C2 (FileID 13,14,15,16,17,18,19,20,21,22,23,24)
#> Reading ./tmp/Data_Normalized_cellType_ID5_Panel1_TP1.fcs_fsom3.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID5_Panel1_TP2.fcs_fsom3.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID5_Panel1_TP3.fcs_fsom3.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID6_Panel1_TP1.fcs_fsom3.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID6_Panel1_TP2.fcs_fsom3.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID6_Panel1_TP3.fcs_fsom3.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID7_Panel1_TP1.fcs_fsom3.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID7_Panel1_TP2.fcs_fsom3.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID7_Panel1_TP3.fcs_fsom3.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID8_Panel1_TP1.fcs_fsom3.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID8_Panel1_TP2.fcs_fsom3.fcs
#> Reading ./tmp/Data_Normalized_cellType_ID8_Panel1_TP3.fcs_fsom3.fcs
#> Computing Splines
#>   C1
#>   C2
```

``` r
saveRDS(object = model3_cellStateMarkersP1, file = "RDS/model3_cellStateMarkersP1.rds")
```

By default, the FlowSOM clustering will be done with the `channels`
listed in the `CytoNorm.train()` call. However, it is possible to
provide other channels for the FlowSOM clustering by listing them at the
`colsToUse` argument within the `FlowSOM.params`. Note that in the
workflow of this paper, CytoNorm will again reuse the previous FlowSOM
model, as discussed before.

## Apply the CytoNorm model

``` r
CytoNorm.normalize(model = model3_cellStateMarkersP1,
                   files = c(files_P1_C1_norm, files_P1_C2_norm),
                   labels = c(rep(x = "C1",times = length(files_P1_C1_norm)),
                              rep(x = "C2",times = length(files_P1_C2_norm))),
                   transformList = NULL,
                   verbose = TRUE,
                   prefix = "",
                   transformList.reverse = NULL, 
                   outputDir = "Data/Normalized_cellState")
#> Splitting Data/Normalized_cellType/ID1_Panel1_TP1.fcs
#> Splitting Data/Normalized_cellType/ID1_Panel1_TP2.fcs
#> Splitting Data/Normalized_cellType/ID1_Panel1_TP3.fcs
#> Splitting Data/Normalized_cellType/ID2_Panel1_TP1.fcs
#> Splitting Data/Normalized_cellType/ID2_Panel1_TP2.fcs
#> Splitting Data/Normalized_cellType/ID2_Panel1_TP3.fcs
#> Splitting Data/Normalized_cellType/ID3_Panel1_TP1.fcs
#> Splitting Data/Normalized_cellType/ID3_Panel1_TP2.fcs
#> Splitting Data/Normalized_cellType/ID3_Panel1_TP3.fcs
#> Splitting Data/Normalized_cellType/ID4_Panel1_TP1.fcs
#> Splitting Data/Normalized_cellType/ID4_Panel1_TP2.fcs
#> Splitting Data/Normalized_cellType/ID4_Panel1_TP3.fcs
#> Splitting Data/Normalized_cellType/ID5_Panel1_TP1.fcs
#> Splitting Data/Normalized_cellType/ID5_Panel1_TP2.fcs
#> Splitting Data/Normalized_cellType/ID5_Panel1_TP3.fcs
#> Splitting Data/Normalized_cellType/ID6_Panel1_TP1.fcs
#> Splitting Data/Normalized_cellType/ID6_Panel1_TP2.fcs
#> Splitting Data/Normalized_cellType/ID6_Panel1_TP3.fcs
#> Splitting Data/Normalized_cellType/ID7_Panel1_TP1.fcs
#> Splitting Data/Normalized_cellType/ID7_Panel1_TP2.fcs
#> Splitting Data/Normalized_cellType/ID7_Panel1_TP3.fcs
#> Splitting Data/Normalized_cellType/ID8_Panel1_TP1.fcs
#> Splitting Data/Normalized_cellType/ID8_Panel1_TP2.fcs
#> Splitting Data/Normalized_cellType/ID8_Panel1_TP3.fcs
#> Warning in FlowSOM::NewData(fsom, ff): 566 cells (0.83%) seem far from their cluster centers.
#> Processing cluster 1
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID1_Panel1_TP1.fcs_fsom1.fcs (C1)
#> Normalizing C1
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID1_Panel1_TP2.fcs_fsom1.fcs (C1)
#> Normalizing C1
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID1_Panel1_TP3.fcs_fsom1.fcs (C1)
#> Normalizing C1
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID2_Panel1_TP1.fcs_fsom1.fcs (C1)
#> Normalizing C1
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID2_Panel1_TP2.fcs_fsom1.fcs (C1)
#> Normalizing C1
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID2_Panel1_TP3.fcs_fsom1.fcs (C1)
#> Normalizing C1
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID3_Panel1_TP1.fcs_fsom1.fcs (C1)
#> Normalizing C1
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID3_Panel1_TP2.fcs_fsom1.fcs (C1)
#> Normalizing C1
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID3_Panel1_TP3.fcs_fsom1.fcs (C1)
#> Normalizing C1
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID4_Panel1_TP1.fcs_fsom1.fcs (C1)
#> Normalizing C1
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID4_Panel1_TP2.fcs_fsom1.fcs (C1)
#> Normalizing C1
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID4_Panel1_TP3.fcs_fsom1.fcs (C1)
#> Normalizing C1
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID5_Panel1_TP1.fcs_fsom1.fcs (C2)
#> Normalizing C2
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID5_Panel1_TP2.fcs_fsom1.fcs (C2)
#> Normalizing C2
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID5_Panel1_TP3.fcs_fsom1.fcs (C2)
#> Normalizing C2
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID6_Panel1_TP1.fcs_fsom1.fcs (C2)
#> Normalizing C2
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID6_Panel1_TP2.fcs_fsom1.fcs (C2)
#> Normalizing C2
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID6_Panel1_TP3.fcs_fsom1.fcs (C2)
#> Normalizing C2
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID7_Panel1_TP1.fcs_fsom1.fcs (C2)
#> Normalizing C2
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID7_Panel1_TP2.fcs_fsom1.fcs (C2)
#> Normalizing C2
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID7_Panel1_TP3.fcs_fsom1.fcs (C2)
#> Normalizing C2
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID8_Panel1_TP1.fcs_fsom1.fcs (C2)
#> Normalizing C2
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID8_Panel1_TP2.fcs_fsom1.fcs (C2)
#> Normalizing C2
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID8_Panel1_TP3.fcs_fsom1.fcs (C2)
#> Normalizing C2
#> Processing cluster 2
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID1_Panel1_TP1.fcs_fsom2.fcs (C1)
#> Normalizing C1
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID1_Panel1_TP2.fcs_fsom2.fcs (C1)
#> Normalizing C1
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID1_Panel1_TP3.fcs_fsom2.fcs (C1)
#> Normalizing C1
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID2_Panel1_TP1.fcs_fsom2.fcs (C1)
#> Normalizing C1
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID2_Panel1_TP2.fcs_fsom2.fcs (C1)
#> Normalizing C1
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID2_Panel1_TP3.fcs_fsom2.fcs (C1)
#> Normalizing C1
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID3_Panel1_TP1.fcs_fsom2.fcs (C1)
#> Normalizing C1
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID3_Panel1_TP2.fcs_fsom2.fcs (C1)
#> Normalizing C1
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID3_Panel1_TP3.fcs_fsom2.fcs (C1)
#> Normalizing C1
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID4_Panel1_TP1.fcs_fsom2.fcs (C1)
#> Normalizing C1
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID4_Panel1_TP2.fcs_fsom2.fcs (C1)
#> Normalizing C1
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID4_Panel1_TP3.fcs_fsom2.fcs (C1)
#> Normalizing C1
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID5_Panel1_TP1.fcs_fsom2.fcs (C2)
#> Normalizing C2
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID5_Panel1_TP2.fcs_fsom2.fcs (C2)
#> Normalizing C2
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID5_Panel1_TP3.fcs_fsom2.fcs (C2)
#> Normalizing C2
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID6_Panel1_TP1.fcs_fsom2.fcs (C2)
#> Normalizing C2
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID6_Panel1_TP2.fcs_fsom2.fcs (C2)
#> Normalizing C2
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID6_Panel1_TP3.fcs_fsom2.fcs (C2)
#> Normalizing C2
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID7_Panel1_TP1.fcs_fsom2.fcs (C2)
#> Normalizing C2
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID7_Panel1_TP2.fcs_fsom2.fcs (C2)
#> Normalizing C2
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID7_Panel1_TP3.fcs_fsom2.fcs (C2)
#> Normalizing C2
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID8_Panel1_TP1.fcs_fsom2.fcs (C2)
#> Normalizing C2
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID8_Panel1_TP2.fcs_fsom2.fcs (C2)
#> Normalizing C2
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID8_Panel1_TP3.fcs_fsom2.fcs (C2)
#> Normalizing C2
#> Processing cluster 3
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID1_Panel1_TP1.fcs_fsom3.fcs (C1)
#> Normalizing C1
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID1_Panel1_TP2.fcs_fsom3.fcs (C1)
#> Normalizing C1
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID1_Panel1_TP3.fcs_fsom3.fcs (C1)
#> Normalizing C1
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID2_Panel1_TP1.fcs_fsom3.fcs (C1)
#> Normalizing C1
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID2_Panel1_TP2.fcs_fsom3.fcs (C1)
#> Normalizing C1
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID2_Panel1_TP3.fcs_fsom3.fcs (C1)
#> Normalizing C1
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID3_Panel1_TP1.fcs_fsom3.fcs (C1)
#> Normalizing C1
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID3_Panel1_TP2.fcs_fsom3.fcs (C1)
#> Normalizing C1
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID3_Panel1_TP3.fcs_fsom3.fcs (C1)
#> Normalizing C1
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID4_Panel1_TP1.fcs_fsom3.fcs (C1)
#> Normalizing C1
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID4_Panel1_TP2.fcs_fsom3.fcs (C1)
#> Normalizing C1
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID4_Panel1_TP3.fcs_fsom3.fcs (C1)
#> Normalizing C1
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID5_Panel1_TP1.fcs_fsom3.fcs (C2)
#> Normalizing C2
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID5_Panel1_TP2.fcs_fsom3.fcs (C2)
#> Normalizing C2
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID5_Panel1_TP3.fcs_fsom3.fcs (C2)
#> Normalizing C2
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID6_Panel1_TP1.fcs_fsom3.fcs (C2)
#> Normalizing C2
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID6_Panel1_TP2.fcs_fsom3.fcs (C2)
#> Normalizing C2
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID6_Panel1_TP3.fcs_fsom3.fcs (C2)
#> Normalizing C2
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID7_Panel1_TP1.fcs_fsom3.fcs (C2)
#> Normalizing C2
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID7_Panel1_TP2.fcs_fsom3.fcs (C2)
#> Normalizing C2
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID7_Panel1_TP3.fcs_fsom3.fcs (C2)
#> Normalizing C2
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID8_Panel1_TP1.fcs_fsom3.fcs (C2)
#> Normalizing C2
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID8_Panel1_TP2.fcs_fsom3.fcs (C2)
#> Normalizing C2
#>   Data/Normalized_cellState/Data_Normalized_cellType_ID8_Panel1_TP3.fcs_fsom3.fcs (C2)
#> Normalizing C2
#> Rebuilding Data/Normalized_cellType/ID1_Panel1_TP1.fcs
#> Rebuilding Data/Normalized_cellType/ID1_Panel1_TP2.fcs
#> Rebuilding Data/Normalized_cellType/ID1_Panel1_TP3.fcs
#> Rebuilding Data/Normalized_cellType/ID2_Panel1_TP1.fcs
#> Rebuilding Data/Normalized_cellType/ID2_Panel1_TP2.fcs
#> Rebuilding Data/Normalized_cellType/ID2_Panel1_TP3.fcs
#> Rebuilding Data/Normalized_cellType/ID3_Panel1_TP1.fcs
#> Rebuilding Data/Normalized_cellType/ID3_Panel1_TP2.fcs
#> Rebuilding Data/Normalized_cellType/ID3_Panel1_TP3.fcs
#> Rebuilding Data/Normalized_cellType/ID4_Panel1_TP1.fcs
#> Rebuilding Data/Normalized_cellType/ID4_Panel1_TP2.fcs
#> Rebuilding Data/Normalized_cellType/ID4_Panel1_TP3.fcs
#> Rebuilding Data/Normalized_cellType/ID5_Panel1_TP1.fcs
#> Rebuilding Data/Normalized_cellType/ID5_Panel1_TP2.fcs
#> Rebuilding Data/Normalized_cellType/ID5_Panel1_TP3.fcs
#> Rebuilding Data/Normalized_cellType/ID6_Panel1_TP1.fcs
#> Rebuilding Data/Normalized_cellType/ID6_Panel1_TP2.fcs
#> Rebuilding Data/Normalized_cellType/ID6_Panel1_TP3.fcs
#> Rebuilding Data/Normalized_cellType/ID7_Panel1_TP1.fcs
#> Rebuilding Data/Normalized_cellType/ID7_Panel1_TP2.fcs
#> Rebuilding Data/Normalized_cellType/ID7_Panel1_TP3.fcs
#> Rebuilding Data/Normalized_cellType/ID8_Panel1_TP1.fcs
#> Rebuilding Data/Normalized_cellType/ID8_Panel1_TP2.fcs
#> Rebuilding Data/Normalized_cellType/ID8_Panel1_TP3.fcs
```

## Evaluate the quality

### Density plots

``` r
# List files
files_P1_C1_norm_norm <- list.files(path = "Data/Normalized_cellState", 
                                    pattern = "ID[1-4]_Panel1_TP..fcs", full.names = TRUE)
files_P1_C2_norm_norm <- list.files(path = "Data/Normalized_cellState", 
                                    pattern = "ID[5-8]_Panel1_TP..fcs", full.names = TRUE)

# Make plots
p <- plotDensities(input = list("P1_C1" = files_P1_C1_norm,
                                "P1_C2" = files_P1_C2_norm,
                                "P1_C1_norm" = files_P1_C1_norm_norm,
                                "P1_C2_norm" = files_P1_C2_norm_norm),
                   channels = P1_cellStateChannels,
                   colors = c("P1_C1" = "#900002","P1_C2" = "#FF3437"), 
                   model = model3_cellStateMarkersP1)
#> Reading Data/Normalized_cellType/ID1_Panel1_TP1.fcs
#> Reading Data/Normalized_cellType/ID1_Panel1_TP2.fcs
#> Reading Data/Normalized_cellType/ID1_Panel1_TP3.fcs
#> Reading Data/Normalized_cellType/ID2_Panel1_TP1.fcs
#> Reading Data/Normalized_cellType/ID2_Panel1_TP2.fcs
#> Reading Data/Normalized_cellType/ID2_Panel1_TP3.fcs
#> Reading Data/Normalized_cellType/ID3_Panel1_TP1.fcs
#> Reading Data/Normalized_cellType/ID3_Panel1_TP2.fcs
#> Reading Data/Normalized_cellType/ID3_Panel1_TP3.fcs
#> Reading Data/Normalized_cellType/ID4_Panel1_TP1.fcs
#> Reading Data/Normalized_cellType/ID4_Panel1_TP2.fcs
#> Reading Data/Normalized_cellType/ID4_Panel1_TP3.fcs
#> Reading Data/Normalized_cellState/ID1_Panel1_TP1.fcs
#> Reading Data/Normalized_cellState/ID1_Panel1_TP2.fcs
#> Reading Data/Normalized_cellState/ID1_Panel1_TP3.fcs
#> Reading Data/Normalized_cellState/ID2_Panel1_TP1.fcs
#> Reading Data/Normalized_cellState/ID2_Panel1_TP2.fcs
#> Reading Data/Normalized_cellState/ID2_Panel1_TP3.fcs
#> Reading Data/Normalized_cellState/ID3_Panel1_TP1.fcs
#> Reading Data/Normalized_cellState/ID3_Panel1_TP2.fcs
#> Reading Data/Normalized_cellState/ID3_Panel1_TP3.fcs
#> Reading Data/Normalized_cellState/ID4_Panel1_TP1.fcs
#> Reading Data/Normalized_cellState/ID4_Panel1_TP2.fcs
#> Reading Data/Normalized_cellState/ID4_Panel1_TP3.fcs
#> Reading Data/Normalized_cellType/ID5_Panel1_TP1.fcs
#> Reading Data/Normalized_cellType/ID5_Panel1_TP2.fcs
#> Reading Data/Normalized_cellType/ID5_Panel1_TP3.fcs
#> Reading Data/Normalized_cellType/ID6_Panel1_TP1.fcs
#> Reading Data/Normalized_cellType/ID6_Panel1_TP2.fcs
#> Reading Data/Normalized_cellType/ID6_Panel1_TP3.fcs
#> Reading Data/Normalized_cellType/ID7_Panel1_TP1.fcs
#> Reading Data/Normalized_cellType/ID7_Panel1_TP2.fcs
#> Reading Data/Normalized_cellType/ID7_Panel1_TP3.fcs
#> Reading Data/Normalized_cellType/ID8_Panel1_TP1.fcs
#> Reading Data/Normalized_cellType/ID8_Panel1_TP2.fcs
#> Reading Data/Normalized_cellType/ID8_Panel1_TP3.fcs
#> Reading Data/Normalized_cellState/ID5_Panel1_TP1.fcs
#> Reading Data/Normalized_cellState/ID5_Panel1_TP2.fcs
#> Reading Data/Normalized_cellState/ID5_Panel1_TP3.fcs
#> Reading Data/Normalized_cellState/ID6_Panel1_TP1.fcs
#> Reading Data/Normalized_cellState/ID6_Panel1_TP2.fcs
#> Reading Data/Normalized_cellState/ID6_Panel1_TP3.fcs
#> Reading Data/Normalized_cellState/ID7_Panel1_TP1.fcs
#> Reading Data/Normalized_cellState/ID7_Panel1_TP2.fcs
#> Reading Data/Normalized_cellState/ID7_Panel1_TP3.fcs
#> Reading Data/Normalized_cellState/ID8_Panel1_TP1.fcs
#> Reading Data/Normalized_cellState/ID8_Panel1_TP2.fcs
#> Reading Data/Normalized_cellState/ID8_Panel1_TP3.fcs
#> [1] "original"
#> Warning in FlowSOM::NewData(model$fsom, as.matrix(dfs[[type]][model$fsom$map$colsUsed])): 1990 cells (1.16%) seem far from their cluster centers.
#> [1] "normalized"
```

``` r

# Arrange figure
p_ <- ggarrange(ggarrange(plotlist = p[1:length(p)-1], 
                          ncol = (length(p)-1)/(2*length(P1_cellStateChannels)), 
                          nrow = 2*length(P1_cellStateChannels)),
                widths = c(1,3),
                p$legend, nrow = 2, heights = c(10,1))
print(p_)
```

![](CytoNorm_channels_without_clustering_files/figure-gfm/Evaluate%20densities-1.png)<!-- -->

### Spline plots

``` r
plotSplines(model = model3_cellStateMarkersP1, channels = P1_cellStateChannels, groupClusters = TRUE)
#> $C1
```

![](CytoNorm_channels_without_clustering_files/figure-gfm/Evaluate%20splines-1.png)<!-- -->

    #> 
    #> $C2
    #> Warning: Removed 65 rows containing missing values or values outside the scale range (`geom_line()`).

![](CytoNorm_channels_without_clustering_files/figure-gfm/Evaluate%20splines-2.png)<!-- -->

### EMD and MAD

#### Make aggregates and get aggregate manual labels

``` r
dir.create("tmp/before")
#> Warning in dir.create("tmp/before"): 'tmp/before' already exists
```

``` r
dir.create("tmp/after")
#> Warning in dir.create("tmp/after"): 'tmp/after' already exists
```

``` r

aggregates <- list("agg_P1_C2" = files_P1_C2)

for (agg in names(aggregates)){
  writeLines(agg)
  files <- aggregates[[agg]]
  set.seed(2023)
  before <- AggregateFlowFrames(fileNames = files, silent = TRUE,
                                cTotal = length(files) * 10000)

  write.FCS(before, paste0("tmp/before/", agg, ".fcs"))  
  after <- before
  
  manual <- c()
  for (i_file in unique(before@exprs[,"File"])){
    ff <- read.FCS(paste0("Data/Normalized_cellType/", basename(aggregates[[agg]][i_file])))
    i_IDs <- before@exprs[before@exprs[,"File"] == i_file,"Original_ID2"]
    after@exprs[before@exprs[,"File"] == i_file,1:(ncol(after@exprs)-3)] <- ff@exprs[i_IDs, ]
    labels <- as.character(manualLabels[[basename(aggregates[[agg]][i_file])]])
    manual <- c(manual, labels[i_IDs])
  }    
  
  manualLabels[[paste0(agg, ".fcs")]] <- factor(manual, levels = levels(manualLabels[[1]]))
  write.FCS(after, paste0("tmp/after/", agg, ".fcs"))
}
#> agg_P1_C2
```

#### Define cell types and files for model evaluation

``` r
cellTypes <- c("unlabeled",
               "All B cells",
               "Transitional B cells",
               "Naive B cells",
               "IgD+ memory B cells",
               "IgM+ memory B cells", 
               "Class switched memory B cells", 
               "Double negative B cells")

models <- list("Model3a_before" = list("files" = c(files_P1_C1_norm, "tmp/before/agg_P1_C1.fcs", 
                                                    "tmp/before/agg_P1_C2.fcs", files_P1_C2_norm),
                                       "channels" = P1_cellStateChannels),
               "Model3a_after" = list("files" = c(files_P1_C1_norm_norm, "tmp/after/agg_P1_C1.fcs", 
                                                   "tmp/after/agg_P1_C2.fcs", files_P1_C2_norm_norm),
                                      "channels" = P1_cellStateChannels))
```

#### Calculate EMD and MAD

``` r
EMDs <- list()
MADs <- list()

for (model in names(models)){
  print(model)
  files <- models[[model]][["files"]]
  channels <- models[[model]][["channels"]]
  
  EMDs[[model]] <- emdEvaluation(files = files,
                                 channels = channels,
                                 manual = manualLabels, return_all = TRUE, 
                                 manualThreshold = 20)
  MADs[[model]] <- CytoNorm:::madEvaluation(files = files[!startsWith(files, "tmp")], 
                                 channels = channels, return_all = TRUE,
                                 manual = manualLabels, transformList = NULL)
}
#> [1] "Model3a_before"
#> [1] "Model3a_after"
```

``` r

EMD_scores <- list()
for (subset in names(models)){
  dist <- EMDs[[subset]]$distances
  EMD_scores[[subset]] <- matrix(NA,
                                 nrow = length(dist), ncol = length(dist[[1]]),
                                 dimnames = list(names(dist),
                                                 names(dist[[1]])))
  for (cellType in names(dist)){
      for (marker in names(dist[[cellType]])){
        EMD_scores[[subset]][cellType, marker] <- dist[[cellType]][[marker]][13, 14]
      }
  }
}
```

#### Make figures

``` r
markerlevels <- NULL
plotlist <- list()

# EMD plot
EMD_before <- EMD_scores[["Model3a_before"]][-2,]
EMD_after <- EMD_scores[["Model3a_after"]][-2,]

EMD_df <- data.frame("before" = c(EMD_before),
                     "after" = c(EMD_after),
                     "feature" = paste(rownames(EMD_before), 
                                       rep(colnames(EMD_before), 
                                           each = nrow(EMD_before)), sep = "_"))
if (is.null(markerlevels)){
  markerlevels <- unique(unique(sub(".*_", "", EMD_df$feature)))
}
EMD_df$marker <- factor(sub(".*_", "", EMD_df$feature), levels = markerlevels)
EMD_df$celltype <- factor(sub("_.*", "", EMD_df$feature), levels = c("AllCells", cellTypes))
max <- max(EMD_df[,1:2], na.rm = T)
p_emd <- ggplot(EMD_df, aes(x = after, y = before)) +
  xlim(0,max) + ylim(0,max) +
  geom_point(aes(color = marker, shape = celltype)) +
  geom_abline() +
  scale_shape_manual(values = c(16, 17, 15, 3, 7, 8, 6)) +
  ggtitle("EMD") +
  theme_minimal()

leg_emd <- get_legend(p_emd)
#> Warning: Removed 42 rows containing missing values or values outside the scale range (`geom_point()`).
```

``` r

p_emd <- p_emd  +
    theme(legend.position = "none")
plotlist[[length(plotlist)+1]] <- p_emd    

names(markerlevels) <- sub(".*<(.*)>.*", "\\1", markerlevels)

# MAD plot
MAD_before <- MADs[["Model3a_before"]][["comparison"]][-2,]
MAD_after <- MADs[["Model3a_after"]][["comparison"]][-2,]
MAD_df <- data.frame("before" = c(MAD_before),
                     "after" = c(MAD_after),
                     "feature" = paste(rownames(MAD_before), 
                                       rep(colnames(MAD_before), 
                                           each = nrow(MAD_before)), sep = "_"))
MAD_df$marker <- factor(markerlevels[sub(".*_", "", MAD_df$feature)], levels = markerlevels)
MAD_df$celltype <- factor(sub("_.*", "", MAD_df$feature), c("AllCells", cellTypes))
max <- max(MAD_df[,1:2], na.rm = T)
p_mad <- ggplot(MAD_df, aes(x = after, y = before)) +
  xlim(0,max) + ylim(0,max) +
  geom_point(aes(color = marker, shape = celltype)) +
  geom_abline() +
  scale_shape_manual(values = c(16, 17, 15, 3, 7, 8, 6)) +
  ggtitle("MAD") +
  theme_minimal()

leg_mad <- get_legend(p_mad)

p_mad <- p_mad  +
    theme(legend.position = "none")

plotlist[[length(plotlist)+1]] <- p_mad

ggarrange(ggarrange(plotlist = plotlist),
          as_ggplot(leg_emd), ncol=2, widths = c(4,1.5))
#> Warning: Removed 42 rows containing missing values or values outside the scale range (`geom_point()`).
```

![](CytoNorm_channels_without_clustering_files/figure-gfm/EMD%20and%20MAD%20figures-1.png)<!-- -->
