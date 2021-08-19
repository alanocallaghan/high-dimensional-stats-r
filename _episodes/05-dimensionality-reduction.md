---
# Please do not edit this file directly; it is auto generated.
# Instead, please edit 05-dimensionality-reduction.md in _episodes_rmd/
title: "Principle component analysis"
author: "GS Robertson"
teaching: 50
exercises: 30
questions:
- Why is PCA useful?
- When is PCA suitable?
- What are some limitations of PCA?
objectives:
-
keypoints:
- 
math: yes
---




# Introduction

Intro to PCA (15 mins: lecture)

* When to use
* What kinds of data/questions exist for which a PCA would be useful
* Limitations of PCA

> ## Challenge 1 
> 
> (5 mins)
>
> Descriptions of three datasets and research questions are given below.
> For which of these is a PCA a useful method of analysis?
>
> 1. Bob has a large dataset (>1m rows) where each row represents a patient 
> admitted to hospital with an infectious respiratory disease. Other data 
> available for each patient includes age, sex, length of time spent in hospital,
> other health conditions, severity of illness (on predefined scale of 1 to 5), 
> and other health-related measures (blood pressure, heart rate etc). 
> Can we predict the type of respiratory illness a patient has been admitted to 
> hospital with using demographic information? Does severity of illness and length
> of stay in hospital vary in patients with different respiratory diseases?
> 2. An online retailer has collected data on user interactions with its online 
> app and has information on the number of times each user interacted with the 
> app, what products they viewed per interaction, and the cost of these products.
> The retailer plans to add a new product to the app and would like to know
> which users to advertise their new product to. 
> 3. Gillian has assayed gene expression levels 
> in 1000 cancer patients and has data from probes targeting different genes in 
> tumour samples from patients. She would like to create a new variable 
> representing different groups of genes to i) find out if genes form subgroups
> based on biological function and ii) use these new variables in a linear
> regression examining how gene expression varies with disease severity.
> 4. All of the above
> 
> > ## Solution
> > Answer: 3
> {: .solution}
{: .challenge}



> ## Challenge 2
> 
> (5 mins)
> Why might it be necessary to standardise continuous variables before
> performing a PCA?  
> 
> 1. To make the results of the PCA interesting
> 2. To ensure that variables with different ranges of values contributes
> equally to analysis
> 3. To allow the feature matrix to be calculated faster, especially in cases
> where there a lot of input variables
> 4. To allow both continuous and categorical variables to be included in the PCA
> 5. All of the above
> 
> > ## Solution
> > Answer: 2
> {: .solution}
{: .challenge}


> ## Challenge 2
> 
> (5 mins)
> 
> Can you think of datasets where it might not be necessary to standardise
> variables? Discuss in groups.
> > ## Solution
> > 1. Datasets which contain continuous variables all measured on the same
> >   scale (e.g. gene expression data or RNA sequencing data). 
> > 2. If you want high-magnitude variables to contribute more to the
> >   resulting PCs.
> {: .solution}
{: .challenge}


# Example of PCA
(code along episode 15 mins)

The USArrests dataset is freely available in R and represents data from 50 US
states containing number of arrests per 100,000 people for three crimes Assault,
Murder, and Rape as well as the percentage of people living in urban areas in
each state (UrbanPop).

Here we will calculate 50 principal component scores for each of the 50 rows
in this dataset, using 4 principal components (one for each of the variables
in the dataset). We want to find out whether these three crime-related variables
can be reduced to a single variable which represents crime levels in different
state. Before PCA can be carried out using these variables need to be 
standardised before PCA can be carried out (number of assaults are likely to be
higher than number of rapes and murders). These variables were standardised to
have a mean of 0 and a standard deviation of 1.

First, we will examine the USArrests data which is part of base R.


~~~
head(USArrests)
~~~
{: .language-r}



~~~
           Murder Assault UrbanPop Rape
Alabama      13.2     236       58 21.2
Alaska       10.0     263       48 44.5
Arizona       8.1     294       80 31.0
Arkansas      8.8     190       50 19.5
California    9.0     276       91 40.6
Colorado      7.9     204       78 38.7
~~~
{: .output}

Notice that names of states are given as row names


~~~
rownames(USArrests)
~~~
{: .language-r}



~~~
 [1] "Alabama"        "Alaska"         "Arizona"        "Arkansas"      
 [5] "California"     "Colorado"       "Connecticut"    "Delaware"      
 [9] "Florida"        "Georgia"        "Hawaii"         "Idaho"         
[13] "Illinois"       "Indiana"        "Iowa"           "Kansas"        
[17] "Kentucky"       "Louisiana"      "Maine"          "Maryland"      
[21] "Massachusetts"  "Michigan"       "Minnesota"      "Mississippi"   
[25] "Missouri"       "Montana"        "Nebraska"       "Nevada"        
[29] "New Hampshire"  "New Jersey"     "New Mexico"     "New York"      
[33] "North Carolina" "North Dakota"   "Ohio"           "Oklahoma"      
[37] "Oregon"         "Pennsylvania"   "Rhode Island"   "South Carolina"
[41] "South Dakota"   "Tennessee"      "Texas"          "Utah"          
[45] "Vermont"        "Virginia"       "Washington"     "West Virginia" 
[49] "Wisconsin"      "Wyoming"       
~~~
{: .output}

Compare the variances between these variables


~~~
apply(USArrests, 2, var)
~~~
{: .language-r}



~~~
    Murder    Assault   UrbanPop       Rape 
  18.97047 6945.16571  209.51878   87.72916 
~~~
{: .output}

Note that variance is greatest for Assaults and lowest for Murder. This makes
sense as you would expect there to be fewer cases of murder than other more
common crimes. UrbanPop is a different variable from Murder, Assault and Rape
measuring the percentage of people per state living in an urban area. We need
to scale each of these variables before including them in a PCA analysis to
ensure that differences in variances between variables do not drive the
calculation of principal components. In this example we standardise all four
variables to have a mean of 0 and a standard deviation of 1. We can do this
inside the prcomp function in R, which carries out a PCA with centred (around
mean = 0) and standardised variables (with a standard deviation of 1). The
`prcomp` function carries out a PCA on the input dataset (where the input data
are in the form of a matrix).


~~~
pcaUS <- prcomp(USArrests, scale = TRUE)
~~~
{: .language-r}

The output from pcaUS returns:
* Standard deviations of each principal component
  (i.e. the square roots of the eigenvalues of the covariance/correlation matrix
  as stated in the prcomp helpfile). In this example there are 4 principal
  components (one for each variable in the dataset).
* The matrix of principal component loadings in which the columns show the
  principal component loading vectors. The is refered to as a rotated (n x k) 
  matrix because when this matrix is multiplied with the matrix containing data 
  on the original coordinate system we get the coordinates in the rotated 
  coordinate system. 


~~~
summary(pcaUS)
~~~
{: .language-r}



~~~
Importance of components:
                          PC1    PC2     PC3     PC4
Standard deviation     1.5749 0.9949 0.59713 0.41645
Proportion of Variance 0.6201 0.2474 0.08914 0.04336
Cumulative Proportion  0.6201 0.8675 0.95664 1.00000
~~~
{: .output}

This returns the proportion of variance in the data explained by each of the
(p = 4) principal components. In this example PC1 explains approximately 62%
of variance in the data, PC2 25% of variance, PC3 a further 9% and PC4
approximately 4%.

We can use a screeplot to see how much variation in the data is explained by
each principal component.


~~~
#variance explained
varExp = (100 * pcaUS$sdev ^ 2) / sum(pcaUS$sdev ^ 2)
#calculate percentage variance explained using output from the PCA
varDF = data.frame(Dimensions=1:length(varExp), varExp=varExp)
#create new dataframe with four rows, one for each principal component
~~~
{: .language-r}


~~~
plot(varDF, ylab = "Percentage of variance explained", xlab = "Principal components")
~~~
{: .language-r}

<img src="../fig/rmd-05-unnamed-chunk-8-1.png" title="Alt" alt="Alt" width="432" style="display: block; margin: auto;" />

The screeplot shows that the first principal component explains most of the
variance in the data (>60%) and each subsequent principal component explains
less and less of the total variance. The first two principal components 
explain >70% of variance in the data. But what do these two principal components
mean?

We can better understand what the principal components represent in terms of
the original variables by plotting the first two principal components against
each other and labelling points by US state name. Clusters of points which have
similar principal component scores can be observed using a biplot and the
strength and direction of influence different variables have on the calculation
of the principal component scores can be observed by plotting arrows
representing the loadings onto the graph. A biplot of the first two principal 
components can be created as follows:


~~~
stats::biplot(pcaUS)
~~~
{: .language-r}

<img src="../fig/rmd-05-unnamed-chunk-9-1.png" title="Alt" alt="Alt" width="432" style="display: block; margin: auto;" />
This biplot shows the position of each state on a 2-dimensional plot where
weight of loadings can be observed via the red arrows associated with each
of the variables. The scale = 0 included as an argument in biplot function
ensures that the arrows are scaled to represent the loadings. In this example,
the biplot shows that the variables Assault and Rape are associated with
negative values on PC1 while negative values on PC2 are associated with the
variable UrbanPop. The length of the arrows indicates how much each variable
contributes to the calculation of each principal component. You can see on this
biplot that UrbanPop is the variable that has the most significant contribution
to PC2.   


# Making use of different PCA packages for analysing biological data

(code along episode 10 mins)


PCAtools provides functions for data exploration using PCA and allows the user
to produce high quality figures. Functions to apply different methods for
choosing appropriate numbers of principal components are available. This PCA
package is designed specifically for the analysis of high dimensional biological
data.

We are going to use PCAtools to explore some gene expression microarray data
downloaded from the Gene Expression Omnibus
(https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE2990). A microarray is a
laboratory tool used to detect the expression of multiple different genes at the
same time. Microarrays used to detect DNA sequences are microscope slides
containing thousands of tiny spots in defined positions, in which each spot
contains a DNA sequence encoding a particular gene. These DNA molecules can be
thought of as probes which detect whether or not a particular gene is expressed
in an input sample. To compare samples (e.g. those taken from patients with
different cancer grades) a microarray analysis can be performed which compares
genes expressed in two hybridised samples. The data collected using microarray
analysis can be used to create gene expression profiles, which show simultaneous
changes in the expression of many genes in response to a particular condition or
treatment. The expression of thousands of genes in a sample can be assessed in
this way.

The dataset we will be analysing in this lesson includes two subsets of data: 
* a matrix of gene expression data showing microarray results for different
  probes used to examine gene expression profiles in 91 different breast cancer
  patient samples.
* metadata associated with the gene expression results detailing information
  from patients from whom samples were taken.

To start our analysis we will download the BiocManager and PCAtools packages
from BioConductor. The BiocManager package is used to install packages from
Bioconductor and PCAtools provides functions that can be used to explore data
via PCA and produce useful figures and analysis tools.


~~~
library("PCAtools")
~~~
{: .language-r}



We will now load the microarray breast cancer gene expression data
(and associated metadata) downloaded from the Gene Expression Omnibus
(https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE2990).


~~~
cancer_data <- readRDS(here::here("data/cancer_expression.rds"))
mat <- assay(cancer_data)
~~~
{: .language-r}



~~~
Error in assay(cancer_data): could not find function "assay"
~~~
{: .error}



~~~
metadata <- colData(cancer_data)
~~~
{: .language-r}



~~~
Error in colData(cancer_data): could not find function "colData"
~~~
{: .error}



~~~
head(mat)
~~~
{: .language-r}



~~~
Error in head(mat): object 'mat' not found
~~~
{: .error}



~~~
dim(mat)
~~~
{: .language-r}



~~~
Error in eval(expr, envir, enclos): object 'mat' not found
~~~
{: .error}



~~~
head(metadata)
~~~
{: .language-r}



~~~
Error in head(metadata): object 'metadata' not found
~~~
{: .error}



~~~
stopifnot(all(colnames(mat) == rownames(metadata)))
~~~
{: .language-r}



~~~
Error in is.data.frame(x): object 'mat' not found
~~~
{: .error}

This dataset was produced by a study which examined whether histologic grade of
breast cancer tumours was associated with gene expression profiles of breast
cancers and whether these profiles could be used to improve histologic grading.

The dataset includes microarray data from two studies (GSE31519 and GSE47561)
with samples from breast carcinomas from a total of 91 patients. The dataset
includes information on probes targeting various genes.

The 'mat' dataset contains a matrix of gene expression profiles for each sample.
Rows represent gene expression variables and columns represent samples. The
'metadata' dataset contains the metadata associated with the gene expression
data including the name of the study from which data originate, the age of the
patient from which the sample was taken, whether or not an oestrogen receptor
was involved in their cancer and the grade and size of the cancer for each
sample (represented by rows).

Microarray data are difficult to analyse for several reasons. Firstly, that
they are typically high dimensional and therefore are subject to the same
difficulties associated with analysing high dimensional data outlined above
(i.e. p>n, large numbers of rows, multiple possible response variables, curse
of dimensionality). Secondly, formulating a research question using microarray
data can be difficult, especially if not much is known a priori about which
genes code for particular phenotypes of interest. Finally, exploratory
analysis, which can be used to help formulate research questions and display
relationships, is difficult using microarray data due to the number of
potentially interesting response variables (i.e. expression data from probes
targeting different genes).

If researchers hypothesise that groups of genes may be associated with different
phenotypic characteristics of cancers (e.g. histologic grade, tumour size),
using statistical methods that reduce the number of columns in the microarray
matrix to a smaller number of dimensions representing groups of genes would
help visualise the data and address research questions regarding the effect
different groups of genes have on disease progression.

Using the Bioconductor package PCAtools we will apply a PCA to the cancer gene
expression data, plot the amount of variation in the data explained by each
principal component and plot the most important principal components against
each other as well as understanding what each principal component represents.

> ## Challenge 3
> 
> (5 mins)
> 
> Apply a PCA to the cancer gene expression data using the 'pca' function from 
> PCAtools.  Use the help files in PCAtools to find out about the 'pca' function.
> Remove the lower 20% of principal components from your PCA. As in the example
> using USArrests data above, examine the first 5 rows and columns of rotated
> data and loadings from your PCA.
> 
> > ## Solution
> > FIXME
> {: .solution}
{: .challenge}




~~~
pc <- pca(mat, metadata = metadata)
~~~
{: .language-r}



~~~
Error in is.data.frame(mat): object 'mat' not found
~~~
{: .error}



~~~
# Many PCs explain a very small amount of the total variance in the data
# Remove the lower 20% of PCs with lower variance

pc <- pca(mat, metadata = metadata, removeVar = 0.2)
~~~
{: .language-r}



~~~
Error in is.data.frame(mat): object 'mat' not found
~~~
{: .error}



~~~
#Explore other arguments provided in pca
pc$rotated[1:5, 1:5]
~~~
{: .language-r}



~~~
Error in eval(expr, envir, enclos): object 'pc' not found
~~~
{: .error}



~~~
pc$loadings[1:5, 1:5]
~~~
{: .language-r}



~~~
Error in eval(expr, envir, enclos): object 'pc' not found
~~~
{: .error}

This function is used to carry out a PCA in PCAtools and uses as inputs a matrix
containing continuous numerical data in which rows are data variables and 
columns are samples, and metadata associated with the matrix in which rows
represent samples and columns represent data variables. The pca function has
options to centre or scale the input data before a PCA is performed, although
in this case gene expression data do not need to be transformed prior to PCA
being carried out as variables are measured on a similar scale (values are
comparable between rows). The output of the pca function includes a lot of
information such as loading values for each variable (probe), principal
components and amount of variance explained by each principal component.

The first principal component is defined as the linear combination of the
original variables (i.e. probes detecting specific genes in the microarray)
that explains the greatest amount of variation. The second principal component
is defined as the linear combination of the original variables that accounts
for the greatest amount of the remaining variation provided that it is
uncorrelated to the first component. Subsequent principal components explain
progressively smaller amounts of variation in the data compared with the first
few components. The first few principal components are the most important as
these represent most of the variation in the data and they are the components
which are most often used to graphically examine the results of the PCA.
How much information is explained by the principal components outputted by the
PCA is important to determine how much information is explained by those
components that are used to interpret the data.

- explain rotated data
- explain loadings

As in the example using the USArrests dataset we can use a screeplot to compare
the proportion of variance in the data explained by each principal component.
This allows us to understand how much information in the microarray dataset is
lost by projecting the observations onto the first few principal conponents and
whether these principal components represent a reasonable amount of the
variation. The proportion of variance explained should sum to one.


> ## Challenge 4
> (5 mins)
> 
> Using the screeplot function in PCAtools, create a screeplot to show
> proportion of variance explained by each principal component. Explain the
> output of the screeplot in terms of proportion of variance in data explained
> by each principal component.
> 
> > ## Solution
> > FIXME
> {: .solution}
{: .challenge}


~~~
screeplot(pc, axisLabSize = 5, titleLabSize = 8)
~~~
{: .language-r}



~~~
Error in getComponents(pcaobj): object 'pc' not found
~~~
{: .error}

Note that first principal component (PC1) explains more variation than other
principal components (which is always the case in PCA). The screeplot shows
that the first principal component only explains ~33% of the total variation
in the micrarray data and many principal components explain very little
variation. The red line shows the cumulative percentage of explained variation
with increasing principal components. Note that in this case 18 principal
components are needed to explain over 75% of variation in the data. This is
not an unusual results for complex biological datasets including genetic
information as clear relationships between groups is sometimes difficult to
observe in the data. The screeplot shows that using a PCA we have reduced
91 predictors to 18 in order to explain a significant amount of variation in
the data. See additional arguments in screeplot function for improving the
appearance of the plot.

There are no clear guidelines on how many principal components should be
included in PCA: your choice depends on the total variability of the data and
the size of the dataset. We often look at the `elbow’ on the screeplot above
as an indicator that the addition of principal components does not drastically
contribute to explain the remaining variance or choose an arbitory cut off for
proportion of variance explained.

Once the most important principal components have been identified using the
screeplot, these can be explored in more detail by plotting principal
components against each other and highlighting points based on variables in
the metadata. This will allow any potential clustering of points according to
demographic or phenotypic variables to be seen.

We can use these plots, called biplots, to look for patterns in the output from
the PCA. 


> ## Challenge 5
> 
> (5 mins)
> 
> Create a biplot of the first two principal components from your PCA (using
> biplot function in PCAtools - see helpfile for arguments) and determine
> whether samples cluster based on variables in metadata. Explain you results.
> 
> > ## Solution
> > 
> > 
> > ~~~
> > biplot(pc, lab = NULL)
> > ~~~
> > {: .language-r}
> > 
> > 
> > 
> > ~~~
> > Error in biplot(pc, lab = NULL): object 'pc' not found
> > ~~~
> > {: .error}
> > 
> > 
> > 
> > ~~~
> > #Find genes associated with probe labels on loadings
> > #This may be included as part of the challenge?
> > ~~~
> > {: .language-r}
> {: .solution}
{: .challenge}


The biplot shows the position of patient samples relative to PC1 and PC2 in
2-dimensional plot. Note that two groups are apparent along the PC1 axis
according to expressions of different genes while no separation can be seem
along the PC2 axis. Labels of patient samples are automatically added in the
biplot. The weight of loadings (i.e. how much each loading contributes to each
PC) can be added to the plot. Labels for each sample are added by default, by
can be removed if there is too much overlap in names. Sizes of labels, points
and axes can be changed (see help file).


~~~
biplot(pc, lab = rownames(pc$metadata), pointSize=1, labSize=1)
~~~
{: .language-r}



~~~
Error in biplot(pc, lab = rownames(pc$metadata), pointSize = 1, labSize = 1): object 'pc' not found
~~~
{: .error}

We can see from this plot that there appear to be two separate groups of points
that separate on the PC1 axis, but that no other grouping is apparent on other
PC axes.




> ## Challenge 6
> 
> (5 mins)
> 
> Use 'colby' and 'lab' arguments in biplot to explore whether these two groups
> may cluster by Age or by whether or not the sample expresses the Estrogen
> Receptor gene (ER+ or ER-).
> 
> > ## Solution
> > 
> > 
> > ~~~
> >   biplot(pc,
> >     lab = paste0(pc$metadata$Age,'years'),
> >     colby = 'ER',
> >     hline = 0, vline = 0,
> >     legendPosition = 'right')
> > ~~~
> > {: .language-r}
> > 
> > 
> > 
> > ~~~
> > Error in biplot(pc, lab = paste0(pc$metadata$Age, "years"), colby = "ER", : object 'pc' not found
> > ~~~
> > {: .error}
> {: .solution}
{: .challenge}


It appears that one cluster has more ER+ samples than the other group.

So far we have only looked at a biplot of PC1 versus PC2 which only gives part
of the picture. The pairplots function in PCAtools can be used to create
multiple biplots including different principal components.


~~~
pairsplot(pc)
~~~
{: .language-r}



~~~
Error in getComponents(pcaobj, seq_len(5)): object 'pc' not found
~~~
{: .error}

Use components to define which principal components will be included in the
plot. Default is 5 PCs.

# Summary
(10 mins)

* Definition of PCA and when to use
* Output from PCA and how to interpret
* Next steps: using PCs in further analysis (e.g. linear regression)