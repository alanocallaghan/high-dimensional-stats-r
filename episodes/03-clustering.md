---
# Please do not edit this file directly; it is auto generated.
# Instead, please edit 03-clustering.md in _episodes_rmd/
title: "High dimensional regression"
teaching: 0
exercises: 0
questions:
- "How can we apply regression methods in a high-dimensional setting?"
objectives:
- "Perform and critically analyse high dimensional regression."
keypoints:
- "Multiple testing correction can enable us to account for many null hypothesis
    significance tests while retaining power."
math: yes
---




R code in RMarkdown (with output):


~~~
rnorm(5)
~~~
{: .language-r}



~~~
[1] -0.5712819 -0.5747766 -0.4657729  2.0712292 -0.9608441
~~~
{: .output}

$\LaTeX$ inline, and in blocks:


$$
    \exp(i\pi) = -1
$$



~~~
## preamble
library("scRNAseq")
~~~
{: .language-r}



~~~
Loading required package: SingleCellExperiment
~~~
{: .output}



~~~
Loading required package: SummarizedExperiment
~~~
{: .output}



~~~
Loading required package: MatrixGenerics
~~~
{: .output}



~~~
Loading required package: matrixStats
~~~
{: .output}



~~~

Attaching package: 'MatrixGenerics'
~~~
{: .output}



~~~
The following objects are masked from 'package:matrixStats':

    colAlls, colAnyNAs, colAnys, colAvgsPerRowSet, colCollapse,
    colCounts, colCummaxs, colCummins, colCumprods, colCumsums,
    colDiffs, colIQRDiffs, colIQRs, colLogSumExps, colMadDiffs,
    colMads, colMaxs, colMeans2, colMedians, colMins, colOrderStats,
    colProds, colQuantiles, colRanges, colRanks, colSdDiffs, colSds,
    colSums2, colTabulates, colVarDiffs, colVars, colWeightedMads,
    colWeightedMeans, colWeightedMedians, colWeightedSds,
    colWeightedVars, rowAlls, rowAnyNAs, rowAnys, rowAvgsPerColSet,
    rowCollapse, rowCounts, rowCummaxs, rowCummins, rowCumprods,
    rowCumsums, rowDiffs, rowIQRDiffs, rowIQRs, rowLogSumExps,
    rowMadDiffs, rowMads, rowMaxs, rowMeans2, rowMedians, rowMins,
    rowOrderStats, rowProds, rowQuantiles, rowRanges, rowRanks,
    rowSdDiffs, rowSds, rowSums2, rowTabulates, rowVarDiffs, rowVars,
    rowWeightedMads, rowWeightedMeans, rowWeightedMedians,
    rowWeightedSds, rowWeightedVars
~~~
{: .output}



~~~
Loading required package: GenomicRanges
~~~
{: .output}



~~~
Loading required package: stats4
~~~
{: .output}



~~~
Loading required package: BiocGenerics
~~~
{: .output}



~~~
Loading required package: parallel
~~~
{: .output}



~~~

Attaching package: 'BiocGenerics'
~~~
{: .output}



~~~
The following objects are masked from 'package:parallel':

    clusterApply, clusterApplyLB, clusterCall, clusterEvalQ,
    clusterExport, clusterMap, parApply, parCapply, parLapply,
    parLapplyLB, parRapply, parSapply, parSapplyLB
~~~
{: .output}



~~~
The following objects are masked from 'package:stats':

    IQR, mad, sd, var, xtabs
~~~
{: .output}



~~~
The following objects are masked from 'package:base':

    anyDuplicated, append, as.data.frame, basename, cbind, colnames,
    dirname, do.call, duplicated, eval, evalq, Filter, Find, get, grep,
    grepl, intersect, is.unsorted, lapply, Map, mapply, match, mget,
    order, paste, pmax, pmax.int, pmin, pmin.int, Position, rank,
    rbind, Reduce, rownames, sapply, setdiff, sort, table, tapply,
    union, unique, unsplit, which.max, which.min
~~~
{: .output}



~~~
Loading required package: S4Vectors
~~~
{: .output}



~~~

Attaching package: 'S4Vectors'
~~~
{: .output}



~~~
The following object is masked from 'package:base':

    expand.grid
~~~
{: .output}



~~~
Loading required package: IRanges
~~~
{: .output}



~~~
Loading required package: GenomeInfoDb
~~~
{: .output}



~~~
Loading required package: Biobase
~~~
{: .output}



~~~
Welcome to Bioconductor

    Vignettes contain introductory material; view with
    'browseVignettes()'. To cite Bioconductor, see
    'citation("Biobase")', and for packages 'citation("pkgname")'.
~~~
{: .output}



~~~

Attaching package: 'Biobase'
~~~
{: .output}



~~~
The following object is masked from 'package:MatrixGenerics':

    rowMedians
~~~
{: .output}



~~~
The following objects are masked from 'package:matrixStats':

    anyMissing, rowMedians
~~~
{: .output}



~~~
library("mixtools")
~~~
{: .language-r}



~~~
mixtools package, version 1.2.0, Released 2020-02-05
This package is based upon work supported by the National Science Foundation under Grant No. SES-0518772.
~~~
{: .output}



~~~
library("irlba")
~~~
{: .language-r}



~~~
Loading required package: Matrix
~~~
{: .output}



~~~

Attaching package: 'Matrix'
~~~
{: .output}



~~~
The following object is masked from 'package:S4Vectors':

    expand
~~~
{: .output}



~~~
library("scater")
~~~
{: .language-r}



~~~
Loading required package: ggplot2
~~~
{: .output}



~~~
library("scuttle")
library("scran")
library("Rtsne")
library("mclust")
~~~
{: .language-r}



~~~
Package 'mclust' version 5.4.7
Type 'citation("mclust")' for citing this R package in publications.
~~~
{: .output}



~~~

Attaching package: 'mclust'
~~~
{: .output}



~~~
The following object is masked from 'package:mixtools':

    dmvnorm
~~~
{: .output}



~~~
library("igraph")
~~~
{: .language-r}



~~~

Attaching package: 'igraph'
~~~
{: .output}



~~~
The following object is masked from 'package:scater':

    normalize
~~~
{: .output}



~~~
The following object is masked from 'package:GenomicRanges':

    union
~~~
{: .output}



~~~
The following object is masked from 'package:IRanges':

    union
~~~
{: .output}



~~~
The following object is masked from 'package:S4Vectors':

    union
~~~
{: .output}



~~~
The following objects are masked from 'package:BiocGenerics':

    normalize, path, union
~~~
{: .output}



~~~
The following objects are masked from 'package:stats':

    decompose, spectrum
~~~
{: .output}



~~~
The following object is masked from 'package:base':

    union
~~~
{: .output}



~~~
library("bluster")
~~~
{: .language-r}



~~~

Attaching package: 'bluster'
~~~
{: .output}



~~~
The following objects are masked from 'package:scran':

    neighborsToKNNGraph, neighborsToSNNGraph
~~~
{: .output}



~~~
zd <- ZeiselBrainData()
~~~
{: .language-r}



~~~
using temporary cache /tmp/RtmpF4eKu9/BiocFileCache
~~~
{: .output}



~~~
snapshotDate(): 2020-10-27
~~~
{: .output}



~~~
see ?scRNAseq and browseVignettes('scRNAseq') for documentation
~~~
{: .output}



~~~
downloading 1 resources
~~~
{: .output}



~~~
retrieving 1 resource
~~~
{: .output}



~~~
loading from cache
~~~
{: .output}



~~~
see ?scRNAseq and browseVignettes('scRNAseq') for documentation
~~~
{: .output}



~~~
downloading 1 resources
~~~
{: .output}



~~~
retrieving 1 resource
~~~
{: .output}



~~~
loading from cache
~~~
{: .output}



~~~
see ?scRNAseq and browseVignettes('scRNAseq') for documentation
~~~
{: .output}



~~~
downloading 1 resources
~~~
{: .output}



~~~
retrieving 1 resource
~~~
{: .output}



~~~
loading from cache
~~~
{: .output}



~~~
snapshotDate(): 2020-10-27
~~~
{: .output}



~~~
see ?scRNAseq and browseVignettes('scRNAseq') for documentation
~~~
{: .output}



~~~
downloading 1 resources
~~~
{: .output}



~~~
retrieving 1 resource
~~~
{: .output}



~~~
loading from cache
~~~
{: .output}



~~~
zd <- computeSumFactors(zd, cluster=quickCluster(zd))
zd <- logNormCounts(zd)

zd <- runPCA(zd, ncomponents = 15)
zd <- runTSNE(zd)



## k-means

## ideas: vary centers, low iter.max, low nstart
cluster <- kmeans(reducedDim(zd), centers = 7, iter.max = 1000, nstart = 100)
zd$kmeans <- as.character(cluster$cluster)

plotReducedDim(zd, "TSNE", colour_by = "kmeans")
~~~
{: .language-r}

<img src="../fig/rmd-02-unnamed-chunk-3-1.png" title="plot of chunk unnamed-chunk-3" alt="plot of chunk unnamed-chunk-3" width="612" style="display: block; margin: auto;" />

~~~
## model-based
clust <- mclustBIC(reducedDim(zd), modelNames = "VVV")
opt_clust <- which.max(clust)


## graph-based
g <- buildSNNGraph(zd, k=10, use.dimred = 'PCA')
clust <- igraph::cluster_walktrap(g)$membership
reducedDim(zd, "force") <- igraph::layout_with_fr(g)
colLabels(zd) <- factor(clust)
plotReducedDim(zd, colour_by="label", dimred="force")
~~~
{: .language-r}

<img src="../fig/rmd-02-unnamed-chunk-3-2.png" title="plot of chunk unnamed-chunk-3" alt="plot of chunk unnamed-chunk-3" width="612" style="display: block; margin: auto;" />

~~~
# bluster::clusterRows - maybe?

## measures: silhouette, bootstrap
## approx silhouette? purity?
~~~
{: .language-r}


{% include links.md %}
