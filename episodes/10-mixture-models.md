---
# Please do not edit this file directly; it is auto generated.
# Instead, please edit 10-mixture-models.md in _episodes_rmd/
title: "Mixture models"
teaching: 45
exercises: 15
questions:
- "How can we cluster low-dimensional data with a model?"
- "What difficulties does high-dimensional clustering present?"
objectives:
- "Understand the basis of mixture models in a low- and high-dimensional
  setting."
keypoints:
- "Mixture models can be used as a clustering method, to model data with
  heterogeneous characteristics."
- "Mixture models are a 'soft' clustering method."
- "Mixture models in high-dimensional data can be difficult to fit and may not
  be ideal."
math: yes
---






# Introduction

High-dimensional data, especially in biological settings, commonly has
many sources of heterogeneity. Some of these are stochastic variation
arising from measurement error or random differences between organisms. 
In some cases, a known grouping causes this heterogeneity (sex, treatment
groups, etc). In other cases, this heterogeneity arises from the presence of
unknown subgroups in the data. Clustering is a set of techniques that allows
us to discover unknown groupings like this, which we can often use to
discover the nature of the heterogeneity we're investigating.

For example, imagine we observed a variable like cancer invasiveness.
When this has one underlying aetiology (origin/cause), the distribution of
our observations of invasiveness will tend to have one peak or *mode*,
with some spread
around that mode. However, if cancer arises due to different causes in different
groups, then there may be distinct subgroups within the data.
For example, some cancers arise through natural processes, but also due to 
environmental pollutants. Furthermore, cancers that arise in younger people
often have different causes (genetic predispositions) that are different to
the causes for the same types of cancer in older people.

An example of a *multi-modal* distribution like this is shown below:


~~~
set.seed(66)
true_means <- c(-1, 4)
true_sds <- c(2, 1)
aggressiveness <- c(
    rnorm(30, mean = true_means[[1]], sd = true_sds[[1]]),
    rnorm(50, mean = true_means[[2]], sd = true_sds[[1]])
)
hist(aggressiveness, breaks = "FD")
~~~
{: .language-r}

<img src="../fig/rmd-10-mixture-data-1.png" title="Alt" alt="Alt" width="432" style="display: block; margin: auto;" />

These data seem to arise from two different groupings, or two different 
distributions. We can imagine modelling this by fitting two distributions, and
labelling each point as belonging to one or the other distribution.
How can we do that? Well, it might help to think about how
we'd fit a distribution to unimodal data first. It's not uncommon to see data
that's roughly normally distributed. For example, cancer volume in a clinical
trial might be normally distributed:


~~~
set.seed(66)
volume <- rnorm(200)
hist(volume, breaks = "FD")
~~~
{: .language-r}

<img src="../fig/rmd-10-unimodal-1.png" title="Alt" alt="Alt" width="432" style="display: block; margin: auto;" />

For data like these, we could
simply measure the mean and standard deviation of the data using `mean` and
`sd`. However, we might not always
see data that looks exactly normal; we might want to fit a different type of
distribution where the parameters can't be estimated quite so simply.
We can use the concept of likelihood to optimise the parameters of any
distribution.
Specifically, we find the set of parameters (in this case, mean and standard
deviation) that best fit the data.


~~~
set.seed(66)
univar <- rnorm(200)
library("MASS")
opt <- fitdistr(x = univar, densfun = dnorm, start = list(mean = 1, sd = 1))
~~~
{: .language-r}



~~~
Warning in densfun(x, parm[1], parm[2], ...): NaNs produced
Warning in densfun(x, parm[1], parm[2], ...): NaNs produced
Warning in densfun(x, parm[1], parm[2], ...): NaNs produced
~~~
{: .warning}



~~~
fitted_mean <- opt$estimate[["mean"]]
fitted_sd <- opt$estimate[["sd"]]
hist(univar, freq = FALSE, breaks = "FD")
curve(
    dnorm(x, mean = fitted_mean, sd = fitted_sd),
    from = min(univar),
    to = max(univar),
    add = TRUE
)
~~~
{: .language-r}

<img src="../fig/rmd-10-fit-univar-1.png" title="Alt" alt="Alt" width="432" style="display: block; margin: auto;" />



> ## Exercise
> 
> 1. What are the `mean` and `sd` for these
>    data? Are they different to the estimates from `fitdistr`?
> 2. Transform the data using `exp` and fit a log-normal distribution to the 
>    data. Compare these with the 
>    empirical mean and standard deviation of this transformed data.
>    *Hint: try `dlnorm` with parameter names `meanlog` and `sdlog`.*
> 
> > ## Solution
> > 1. 
> >    
> >    ~~~
> >    opt
> >    ~~~
> >    {: .language-r}
> >    
> >    
> >    
> >    ~~~
> >          mean          sd    
> >      0.03869677   0.92072029 
> >     (0.06510476) (0.04603563)
> >    ~~~
> >    {: .output}
> >    
> >    
> >    
> >    ~~~
> >    mean(univar)
> >    ~~~
> >    {: .language-r}
> >    
> >    
> >    
> >    ~~~
> >    [1] 0.03869649
> >    ~~~
> >    {: .output}
> >    
> >    
> >    
> >    ~~~
> >    sd(univar)
> >    ~~~
> >    {: .language-r}
> >    
> >    
> >    
> >    ~~~
> >    [1] 0.9230326
> >    ~~~
> >    {: .output}
> > 2. 
> >    
> >    ~~~
> >    univar_exp <- exp(univar)
> >    opt_exp <- fitdistr(x = univar_exp, densfun = dlnorm, start = list(meanlog = 1, sdlog = 1))
> >    ~~~
> >    {: .language-r}
> >    
> >    
> >    
> >    ~~~
> >    Warning in densfun(x, parm[1], parm[2], ...): NaNs produced
> >    Warning in densfun(x, parm[1], parm[2], ...): NaNs produced
> >    Warning in densfun(x, parm[1], parm[2], ...): NaNs produced
> >    ~~~
> >    {: .warning}
> >    
> >    
> >    
> >    ~~~
> >    opt_exp
> >    ~~~
> >    {: .language-r}
> >    
> >    
> >    
> >    ~~~
> >        meanlog       sdlog   
> >      0.03869677   0.92072029 
> >     (0.06510476) (0.04603563)
> >    ~~~
> >    {: .output}
> >    
> >    
> >    
> >    ~~~
> >    mean(univar_exp)
> >    ~~~
> >    {: .language-r}
> >    
> >    
> >    
> >    ~~~
> >    [1] 1.587429
> >    ~~~
> >    {: .output}
> >    
> >    
> >    
> >    ~~~
> >    sd(univar_exp)
> >    ~~~
> >    {: .language-r}
> >    
> >    
> >    
> >    ~~~
> >    [1] 1.698602
> >    ~~~
> >    {: .output}
> >    
> >    
> >    
> >    ~~~
> >    fitted_mean_log <- opt_exp$estimate[["meanlog"]]
> >    fitted_sd_exp <- opt_exp$estimate[["sdlog"]]
> >    hist(univar_exp, freq = FALSE, breaks = "FD")
> >    ~~~
> >    {: .language-r}
> >    
> >    <img src="../fig/rmd-10-fit-dlnorm-1.png" title="plot of chunk fit-dlnorm" alt="plot of chunk fit-dlnorm" width="432" style="display: block; margin: auto;" />
> >    
> >    ~~~
> >    curve(
> >        dnorm(x, mean = fitted_mean_exp, sd = fitted_sd_exp),
> >        from = min(univar_exp),
> >        to = max(univar_exp),
> >        add = TRUE
> >    )
> >    ~~~
> >    {: .language-r}
> >    
> >    
> >    
> >    ~~~
> >    Error in dnorm(x, mean = fitted_mean_exp, sd = fitted_sd_exp): object 'fitted_mean_exp' not found
> >    ~~~
> >    {: .error}
> {: .solution}
{: .challenge}

## Fitting a mixture model

Now, let's return to the example that looks like a mixture of two distributions.
To fit two different distributions to these data, we can use an algorithm 
call EM, or "expectation-maximisation". This refers to the two steps of the
algorithm.
First, we choose some initial values for the distributions we want to fit.
We can fit any number of distributions to the data, and this number is often 
denoted $k$. In this case, we want to fit two components, so $k=2$.
It's important to note that we don't necessarily have to pick good starting
values here, though it may help. You can see that below our initial starting
"guess" is really bad in this case:


~~~
Warning: Removed 4 rows containing missing values (geom_bar).
~~~
{: .warning}

<img src="../fig/rmd-10-mixture-animation-1.png" title="Alt" alt="Alt" width="432" style="display: block; margin: auto;" />

We then assign each data point to the component that fits them better (this is
the "expectation" step). Then, we maximise the likelihood of the data under
each distribution. That is, we find the best-fitting parameters of the 
distributions for each of the $k$ components. We continue this two-step process
until the algorithm converges -- meaning that the components don't change from 
iteration to iteration. In this simple example, the algorithm converges after 
one iteration, but this won't usually be the case!


~~~
Warning: Removed 4 rows containing missing values (geom_bar).
~~~
{: .warning}

<img src="../fig/rmd-10-mix-converged-1.png" title="Alt" alt="Alt" width="432" style="display: block; margin: auto;" />

The figures shown here were made manually, to be able to step through the 
process. To fit a 2-D mixture model, it's usually not wise to code it yourself,
because people have made very fast and easy-to-use packages to fit mixture 
models. Here's one example:


~~~
set.seed(66)
true_means <- c(-1, 4)
true_sds <- c(2, 1)
aggressiveness <- c(
    rnorm(30, mean = true_means[[1]], sd = true_sds[[1]]),
    rnorm(50, mean = true_means[[2]], sd = true_sds[[1]])
)

library("mixtools")
mix <- normalmixEM2comp(
    aggressiveness,
    lambda = c(0.5, 0.5),
    mu = c(0, 0.1),
    sigsqrd = c(1, 1)
)
~~~
{: .language-r}



~~~
number of iterations= 177 
~~~
{: .output}



~~~
plot(mix, whichplots = 2)
~~~
{: .language-r}

<img src="../fig/rmd-10-fit-mixem-1.png" title="plot of chunk fit-mixem" alt="plot of chunk fit-mixem" width="432" style="display: block; margin: auto;" />

We can also see that the model recovers mean and sd values pretty close to the
ground truth:


~~~
mix$mu
~~~
{: .language-r}



~~~
[1] -1.700932  3.783077
~~~
{: .output}



~~~
true_means
~~~
{: .language-r}



~~~
[1] -1  4
~~~
{: .output}



~~~
mix$sigma
~~~
{: .language-r}



~~~
[1] 1.283686 2.176896
~~~
{: .output}



~~~
true_sds
~~~
{: .language-r}



~~~
[1] 2 1
~~~
{: .output}

> ## Exercise
>
> Try changing the `true_means` and `true_sds` parameters to different values
> and fitting a mixture model to the data.
> 
> How do the results change? At what point is it hard to reliably separate the
> two distributions?
> 
> > ## Solution
> > 
> > If we keep the `true_sds` the same and change the means to be `c(-1, 1)`,
> > a mixture model can't reliably recover the input.
> > 
> > That's because the left, broader distribution centred at `-1` "bleeds into"
> > the right distribution.
> >
> > 
> > ~~~
> > set.seed(66)
> > true_means <- c(-1, 1)
> > true_sds <- c(2, 1)
> > aggressiveness <- c(
> >     rnorm(30, mean = true_means[[1]], sd = true_sds[[1]]),
> >     rnorm(50, mean = true_means[[2]], sd = true_sds[[1]])
> > )
> > 
> > mix <- normalmixEM2comp(
> >     aggressiveness,
> >     lambda = c(0.5, 0.5),
> >     mu = c(0, 0.1),
> >     sigsqrd = c(1, 1)
> > )
> > ~~~
> > {: .language-r}
> > 
> > 
> > 
> > ~~~
> > number of iterations= 152 
> > ~~~
> > {: .output}
> > 
> > 
> > 
> > ~~~
> > plot(mix, whichplots = 2)
> > ~~~
> > {: .language-r}
> > 
> > <img src="../fig/rmd-10-mix-expt-1.png" title="plot of chunk mix-expt" alt="plot of chunk mix-expt" width="432" style="display: block; margin: auto;" />
> > 
> > ~~~
> > mix$mu
> > ~~~
> > {: .language-r}
> > 
> > 
> > 
> > ~~~
> > [1] -0.506124  3.926377
> > ~~~
> > {: .output}
> > 
> > 
> > 
> > ~~~
> > true_means
> > ~~~
> > {: .language-r}
> > 
> > 
> > 
> > ~~~
> > [1] -1  1
> > ~~~
> > {: .output}
> {: .solution}
{: .challenge}


# Mixture models in more than one dimension

Of course, biological data is not usually so one-dimensional! In fact, for
these clustering exercises, we're doing to work with single-cell RNAseq data,
which is often *very* high-dimensional. Commonly, experiments profile the expression
level of 10,000+ genes in thousands of cells. Even after filtering the 
data to remove low quality observations, the dataset we're using in this episode
contains measurements for over 9,000 genes in over 3,000 cells.


~~~
library("SingleCellExperiment")
scrnaseq <- readRDS(here::here("data/scrnaseq.rds"))
dim(scrnaseq)
~~~
{: .language-r}



~~~
[1] 9715 3005
~~~
{: .output}

One way to get a handle on data of this size is to use something we covered
earlier in the course - dimensionality reduction!
Dimensionality reduction allows us to visualise this incredibly complex data
in a small number of dimensions.
In this case, we'll primarily be using PCA. This allows us to compress the data 
by identifying the major axes of variation in the data,
and to run our clustering algorithms on this lower-dimensional data.

The `scater` package has some easy-to-use tools to calculate and plot 
dimensionality reduction for `SummarizedExperiment` objects.
If we plot the first two principal components, we can see that the data points
are spread out roughly continuously, with some clustering.


~~~
library("scater")
scrnaseq <- runPCA(scrnaseq, ncomponents = 15)
plotReducedDim(scrnaseq, "PCA")
~~~
{: .language-r}

<img src="../fig/rmd-10-reddim-1.png" title="plot of chunk reddim" alt="plot of chunk reddim" width="432" style="display: block; margin: auto;" />

You can see from the axis labels that the first two principal components
capture almost 50% of the variation within the data.
For now, we'll work with just these two principal components, since we can
visualise those easily, and they're a quantitative
representation of the underlying data, representing the two largest axes of
variation. For speed, we'll take a random subset of 1/5 of the data.


~~~
set.seed(42)
random_ind <- sample(ncol(scrnaseq), ceiling(ncol(scrnaseq) / 5))
pcs <- reducedDim(scrnaseq, "PCA")[random_ind, 1:2]
plot(pcs)
~~~
{: .language-r}

<img src="../fig/rmd-10-pcs-1.png" title="plot of chunk pcs" alt="plot of chunk pcs" width="432" style="display: block; margin: auto;" />

# Multivariate distributions (distributions of more than one variable)

To fit a mixture model to these first two principal components, we're going
to fit a mixture of multivariate normal distributions. These multivariate
distributions are very similar to a number of univariate normal distributions
combined. For example, if we generate two sets of normally distributed variables
and plot them against each other, we get a "cloud" of points that's roughly 
round with most of the points in the centre:

<img src="../fig/rmd-10-norms-1.png" title="plot of chunk norms" alt="plot of chunk norms" width="432" style="display: block; margin: auto;" />

A multivariate normal distribution can be similar to this, but it models
both variables at once. In fact, in some cases it can basically be identical:

<img src="../fig/rmd-10-mvnorm-1.png" title="plot of chunk mvnorm" alt="plot of chunk mvnorm" width="432" style="display: block; margin: auto;" />

However, it also allows us to model sets of variables that aren't *independent*:

<img src="../fig/rmd-10-mvnormcor-1.png" title="plot of chunk mvnormcor" alt="plot of chunk mvnormcor" width="432" style="display: block; margin: auto;" />

This is useful in a mixture model, because there's no reason to think that
clusters of data will always be best modelled by a ball-shaped distribution.

To fit a 2D mixture model, we can again use the `mixtools` package.
This time, we want the function `mvnormalmixEM`. This is short for 
"multivariate normal mixture model fit with Expectation Maximisation".
We can fit this model to our principal components and see what the model
looks like. We'll set $k=2$ as a starting point.


~~~
mix_sc2 <- mvnormalmixEM(pcs, k = 2)
~~~
{: .language-r}



~~~
number of iterations= 22 
~~~
{: .output}



~~~
plot(mix_sc2, 2, pch = 19, cex = 0.5)
~~~
{: .language-r}

<img src="../fig/rmd-10-mix2-1.png" title="plot of chunk mix2" alt="plot of chunk mix2" width="432" style="display: block; margin: auto;" />

Hmm. Our model has fit the data, but are these the clusters you expected
it to find?

> ## Exercise
>
> Using the same seed (42), fit the same type of model with $k=3$. How different 
> are the results?
> Which do you think is better? Be sure to set the random seed before running
> the model!
>
> Try again with k=3 without resetting the seed. Is this model better or worse?
> Do you think k should be increased more?
> 
> > ## Solution
> > 
> > 
> > ~~~
> > set.seed(42)
> > mix_sc3 <- mvnormalmixEM(pcs, k = 3)
> > ~~~
> > {: .language-r}
> > 
> > 
> > 
> > ~~~
> > number of iterations= 49 
> > ~~~
> > {: .output}
> > 
> > 
> > 
> > ~~~
> > plot(mix_sc3, 2, pch = 19, cex = 0.5)
> > ~~~
> > {: .language-r}
> > 
> > <img src="../fig/rmd-10-mix3-1.png" title="plot of chunk mix3" alt="plot of chunk mix3" width="432" style="display: block; margin: auto;" />
> > 
> > ~~~
> > mix_sc3_2 <- mvnormalmixEM(pcs, k = 3)
> > ~~~
> > {: .language-r}
> > 
> > 
> > 
> > ~~~
> > number of iterations= 183 
> > ~~~
> > {: .output}
> > 
> > 
> > 
> > ~~~
> > plot(mix_sc3_2, 2, pch = 19, cex = 0.5)
> > ~~~
> > {: .language-r}
> > 
> > <img src="../fig/rmd-10-mix3_2-1.png" title="plot of chunk mix3_2" alt="plot of chunk mix3_2" width="432" style="display: block; margin: auto;" />
> {: .solution}
{: .challenge}

You can hopefully see that with real data, clustering can be a bit of a tricky 
business! In fact, even in two dimensions it's not entirely clear what the 
correct clustering is, nor even the true number of clusters.



> ## Mixture shape
> 
> One problem with mixture models with more than one variable is that multivariate
> distributions can be computationally difficult to estimate. 
> To combat this, there's a number of simplifying assumptions we can make.
> For example, if our variables were totally uncorrelared, we might think that 
> all our clusters were normally distributed without any correlation. 
> In our case,
> this clearly isn't true: there's a lot of differng shapes. 
> Alternatively, we could allow the shape to vary, but assume that all clusters
> have the same shape.
> We could also assume that all clusters have the same amount of within-cluster
> variability (meaning they are the same volume).
> 
> The R package [`mclust`](https://cran.r-project.org/web/packages/mclust/vignettes/mclust.html)
> has a number of options to fit mixture models in a very efficient way, and
> to test out different types of assumptions.
> 
> In this case, we're going to avoid any of these assumptions to be as flexible as
> possible. This means that we're allowing the normal distributions to have 
> varying shape, the shape to vary between clusters, and
> the clusters to each have varying amounts of within-cluster variability.
> This is encoded in the setting `modelNames = "VVV"`.
> 
> 
> ~~~
> library("mclust")
> pcs_12 <- reducedDim(scrnaseq, "PCA")[, 1:2]
> clust <- mclustBIC(pcs_12, modelNames = "VVV")
> plot(clust)
> ~~~
> {: .language-r}
> 
> <img src="../fig/rmd-10-mixture-1.png" title="Alt" alt="Alt" width="432" style="display: block; margin: auto;" />
> 
> ~~~
> model <- Mclust(pcs_12, x = clust)
> plot(model, what = "classification")
> ~~~
> {: .language-r}
> 
> <img src="../fig/rmd-10-mixture-2.png" title="Alt" alt="Alt" width="432" style="display: block; margin: auto;" />
> 
> You can probably also see that for 
> very high-dimensional data, the kind of assumption we made with our simulated 
> data (normal distribution) can be difficult to justify. The tails of the 
> clusters especially don't seem to fit a normal distribution very well, and it 
> seems like a distribution with a different shape might fit a bit better here.
> We'll address some of these issues in the next episode!
{: .callout}




> ## t-SNE and UMAP
> 
> t-SNE and UMAP are dimensionality reduction methods which seek to create
> a low-dimensional representation of high-dimensional data, ensuring
> that points which are neighbours (close to each other) in the original
> high-dimensional data are also neighbours in the low-dimensional 
> representation.
> 
> Like MDS, they are stochastic algorithms and aren't quantitative in the way
> that PCA is.
> 
> In contrast to PCA which we've been looking at so far,
> t-SNE and UMAP tend to separate the data into "blobs". This isn't
> necessarily good, and it can be easy to deceive yourself into thinking that
> the blobs made in these plots have meaning that they don't really have.
> 
> While it can be a useful tool when performing clustering in more than two
> dimensions, it's important to remember that the results from a robust 
> cluster analysis may not match what we see in a t-SNE or UMAP plot.
> 
> 
> ~~~
> scrnaseq <- runTSNE(scrnaseq, dimred = "PCA")
> plotReducedDim(scrnaseq, "TSNE")
> ~~~
> {: .language-r}
> 
> <img src="../fig/rmd-10-tsne-1.png" title="plot of chunk tsne" alt="plot of chunk tsne" width="432" style="display: block; margin: auto;" />
{: .callout}

## Further reading

- [Modern statistics for modern biology; Susan Holmes and Wolfgang Huber (Chapter 4)](https://web.stanford.edu/class/bios221/book/Chap-Mixtures.html).

{% include links.md %}