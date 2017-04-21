
# Analysis of Illumina Microarray data with R and Bioconductor 

**Center for Research Informatics, University of Chicago**

April - June 2017; Saturdays 9:00AM - 12:00PM

**Instructor:** Jorge Andrade, Ph.D.


## Learning Objectives

This hands-on tutorial is focused on the analysis of Illumina microarray data using R and Bioconductor, this tutorial assumes that you have previous experience using R for data analysis.

## 1. The Data:

- The paper is available in [Pubmed](https://www.ncbi.nlm.nih.gov/pubmed/23264745)

### Getting the data 



* Observe that the first 6 lines are meatadata, the expression profile of the samples starts in line 7 with the header. Illumina Probe_ids are on rows, and samples on the columns.

## 2. Data analysis workflow:

![workflow](https://raw.githubusercontent.com/MScBiomedicalInformatics/MSIB32500/master/cheatsheets/workflow.png)
The first thing we will need to start working with R and Bioconductor, is to download and install the packages we will need. For this analysis we will use the Bioconductor **lumi** package. The **lumi** package provides an integrated solution for Illumina microarray data analysis, it includes functions for Illumina BeadStudio (GenomeStudio) data input, quality control, BeadArray-specific variance stabilization, normalization and gene annotation at the probe level. Available functions include: **lumiR(), lumiB(), lumiT(), lumiN() and lumiQ()** designed for data input, preprocessing and quality control. 

Downloading and installing **lumi** package:

```{r}
> source("http://bioconductor.org/biocLite.R")
```

We will also need the corresponding **annotation libraries** **lumiMouseAll.db** and **AnnotationDbi**, we can download and install these libraries with the code below:

```{r}
> biocLite("lumiMouseAll.db")
```
## 4. Reading the data

The function **lumiR()** supports directly reading of the Illumina Bead Studio toolkit output from version 1 to version 3. It can automatically detect the BeadStudio output version/format and create a new *LumiBatch* object for it. The **lumiR** function will automatically determine the starting line of the data; columns with header including AVG_Signal and BEAD_STD are required for the LumiBatch object. The sample IDs and sample labels are extracted from the column names of the data file. After reading the data, **lumiR** will *automatically* initialize the **QC slot** of the LumiBatch object by calling **lumiQ.**

To read the data, we will start by defining our working directory:

```{r}
> setwd("/Users/jorgeandrade/Desktop/GSE43221")
> getwd()

```
The folder **GSE43221** contains the **GSE43221_non-normalized_data.txt** file; we will now read/load the file into R using **lumiR**. The annotation file that we just installed **lumiMouseAll.db** is also loaded into the **featureData** of the *LumiBatch* object **x.lumi**

```{r}
> x.lumi <- lumiR('GSE43221_non-normalized_data.txt', lib='lumiMouseAll')
```
Next we will need to create a **.txt** file with the *phenotype information*, please create a tab-delimited file that list all the samples and associated phenotype as follow:

![pheno](https://raw.githubusercontent.com/MScBiomedicalInformatics/MSIB32500/master/cheatsheets/pheno.png)

For your convenience, I have already created that file and it is available [here](https://raw.githubusercontent.com/MScBiomedicalInformatics/MSIB32500/master/data/Phenod.txt) Download this file to your working directory.

###Importing the phenotype data: 
```

Output:

```
                  Species Type Treatment Time Replicate
```
To get a data summary report we simply use:

```{r}
> x.lumi
```

Output:

```
Summary of data information:
```
## 5. Preliminary exploratory analysis

We will now start an exploratory analysis of the data using different methods; this analysis will allow us to easily identify outliers and/or other problems with the samples.

```{r}
```
![box](https://raw.githubusercontent.com/MScBiomedicalInformatics/MSIB32500/master/cheatsheets/boxplot.png)

####These two graphics are showing us: 

We will now explore the correlation of samples before normalization:

Pairwise correlation of TR control group: 

```{r}
> pairs(x.lumi[,c(1,2,3)], smoothScatter=TRUE)
![correlation](https://raw.githubusercontent.com/MScBiomedicalInformatics/MSIB32500/master/cheatsheets/corr.png)

You can continue exploring the correlation of technical replicates (TR) for other groups by changing the values at: *pairs(x.lumi[,c(1,2,3)]*

Next we will create the [MA plot](https://en.wikipedia.org/wiki/MA_plot) for some samples of the LumiBatch object by using plot or **MAplot** functions. MA plots are used to determine if the data needs normalization and to test if the normalization worked. **M** is defined as the log intensity ratio (or difference between log intensities) and **A** is the average log intensity for a dot in the plot.

```{r}
> MAplot(x.lumi[,c(1,2,3)])
```
![MAplot](https://raw.githubusercontent.com/MScBiomedicalInformatics/MSIB32500/master/cheatsheets/MAplot.png)

## 6. Backgroud correction

### Variance-stabilizing transformation
```
Output:

```
Perform vst transformation ...
```
```{r}
trans <- plotVST(lumi.T)
```
![trans](https://raw.githubusercontent.com/MScBiomedicalInformatics/MSIB32500/master/cheatsheets/trans.png)

Now we will compare the VST and log2 transformations

```{r}
> matplot(log2(trans$untransformed), trans$transformed, type='l', main='compare VST and log2 transform', xlab='log2 transformed', ylab='VST transformed')
> abline(a=0, b=1, col=2)
```
![VST](https://raw.githubusercontent.com/MScBiomedicalInformatics/MSIB32500/master/cheatsheets/VST.png)


## 7. Data Normalization
```

We can now plot the density and box plot of our samples, after **RSN** normalization.

```{r}
> plot(lumi.N.Q, what='density')
```
Density After Normalization:
![densityAN](https://raw.githubusercontent.com/MScBiomedicalInformatics/MSIB32500/master/cheatsheets/DensityAN.png)
Box Plot After Normalization:
![boxplotAN](https://raw.githubusercontent.com/MScBiomedicalInformatics/MSIB32500/master/cheatsheets/boxplotAN.png)

Next we will plot the **MAplots** of samples in the control group after normalization:

```{r}
> MAplot(lumi.N.Q[,c(1,2,3)], smoothScatter=TRUE)
```
![MAplotAN](https://raw.githubusercontent.com/MScBiomedicalInformatics/MSIB32500/master/cheatsheets/MAplotAN.png)

##8. Principal Component Analysis


```
Importance of components:
                            PC1      PC2      PC3     PC4      PC5     PC6      PC7      PC8     PC9     PC10
Standard deviation     143.1359 36.41579 24.26316 19.5683 19.04346 17.2648 16.35583 15.73560 14.9494 13.63279
Proportion of Variance   0.7973  0.05161  0.02291  0.0149  0.01411  0.0116  0.01041  0.00964  0.0087  0.00723
Cumulative Proportion    0.7973  0.84889  0.87180  0.8867  0.90082  0.9124  0.92283  0.93246  0.9412  0.94839
                           PC11     PC12     PC13     PC14     PC15     PC16    PC17    PC18    PC19    PC20
Standard deviation     13.28207 12.81103 12.27342 11.52524 10.92886 10.04001 9.86903 9.52206 9.41028 8.26338
Proportion of Variance  0.00687  0.00639  0.00586  0.00517  0.00465  0.00392 0.00379 0.00353 0.00345 0.00266
Cumulative Proportion   0.95526  0.96164  0.96750  0.97267  0.97732  0.98124 0.98503 0.98856 0.99201 0.99467
                          PC21    PC22   PC23      PC24
Standard deviation     7.53919 7.03579 5.5426 6.326e-14
Proportion of Variance 0.00221 0.00193 0.0012 0.000e+00
Cumulative Proportion  0.99688 0.99880 1.0000 1.000e+00


### Plotting the PCA
For this tutorial, we will use the library **rgl**, a 3D Visualization library that uses OpenGL.

* Note: MacOS user will need to download and install: [www.xquartz.org](https://www.xquartz.org)

> library("rgl")
Now we will create an array of different colors to distinguish samples that belong to the same group:

```{r}
```
You should be able to see and interactive 3D PCA plot as bellow:

![PCA](https://raw.githubusercontent.com/MScBiomedicalInformatics/MSIB32500/master/cheatsheets/PCA.png)

The PCA plot shows samples that belong to the same group in similar color. Note WT samples are represented in Red and KO samples in SkyBlue.


##9. Encapsulating the processing steps

* The function **lumiExpresso()** is designed to encapsulate the major functions of Illumina preprocessing. It is organized in a similar way as the function **expresso()** in the affy package. 

* The following code will perform the same processing as the previous multi-steps, and produced the same results as **lumi.N.Q()**.

```{r}
> lumi.N.Q1 <- lumiExpresso(x.lumi, normalize.param=list(method='rsn'))
```
At this point we can save the **“expression matrix”** into a tab delimited text file:

```{r}
> dataMatrix <- exprs(lumi.N.Q)
```

##10.Data filtering

* We will now filter our LumiBatch object for probes with only **“good”** detectable signal (detection p-value smaller that 0.01). 
* Our filtering criteria will be to require detectable signal in at least 80% of our 24 samples:
```
We will then create a list of probeIDs:

```

##11. Identifying Differentially Expressed Genes (DEGs)

For this tutorial, we will proceed to identify differentially expresed genes (DEG) only between control (WT, 3 samples) and 24 hr. of treatment (WT 24hr, 3 samples); subsequent pairwise DEG analysis can be repeated for different groups:
```

```{r}
> head(M6data)

```

```

                   CCE_WT_Ctrl_1 CCE_WT_Ctrl_2 CCE_WT_Ctrl_3 CCE_WT_RA_24hr_1

```
Let us review our phenotype data:

```{r}
> phenod
```


```
Species Type Treatment Time Replicate
```
We will now create a subset of phenod with only the phenotype for our 6 selected samples: 

We will use the limma  (Linear Models for Microarray Data) package from Bioconductor to perform the analysis of diferential gene expression:
```

We will create a two groups comparison design matrix:
```

And fit a linear model for each gene in the expression set M6data given the design matrix:
```

Now we are going to calculate the **differential expression** by **empirical Bayes shrinkage** of the standard errors towards a common value, by computing the moderated t-statistics, moderated F-statistic, and log-odds:
```

Now, we will create (and save) a table with the calculated statistics for the selected group of samples sorted by corrected p-value:
```
Next, we are going to filter genes that have adjusted **p-values** less that 0.001, to create a smaller list of significant genes:
And save those highly significant DEGs in a Excel like format:


```{r}
attach(tab.sig)
sortedDEG <- tab.sig[order(adj.P.Val, logFC),]

```{r}
top20DEGs<-head(sortedDEG,20)
write.table(top20DEGs,file="top20DEGs.xls", quote=FALSE, row.names=T, sep="\t")


Next we are going to create a **heatmap** of the expression of highly significant genes:
![heatmap](https://raw.githubusercontent.com/MScBiomedicalInformatics/MSIB32500/master/cheatsheets/heatmap.png)

##13. Annotation 

The final step in this analysis will be the annotation of the DEGs. First we are going to download and install the libraries we need for the annoation:
```
To list the objects available in this annotation package we can use:

```{r}
```
Output:

```
[1] "lumiMouseAll"              "lumiMouseAll.db"          
```

In general *ProbeIDs* are used to map microarray measurements and probes. However, Illumina’s ProbeIDs can change with different versions, and even between different batches of Illumina microarrays. To solve this problem, **lumi** designed a nucleotide universal identifier **(nuID)**, which encodes the **50mer oligonucleotide sequence** and contains error checking and self-identification code. We will now annotate **nuIDs** with the corresponding **gene** information:

```{r}
> ID <- row.names(tab.sig)
```

Output:

```
[1] "ZUsnIvMeX7f0IdPiAo" "ELiaujnucbuA3t.lKU" "iohWMQGvTB5.TFycp4"
```

```{r}
> Symbol <- getSYMBOL(ID, 'lumiMouseAll')
```

For each Ensembl ID (if we have it), we will now create a hyperlink that goes to the Ensembl genome browser:

```{r}
> Ensembl <- ifelse(Ensembl=="NA", NA,
                        
```

And make a temporary data frame with all those identifiers:

```{r}
> tmp <- data.frame(ID=ID, Symbol=Symbol, Name=Name, logFC=tab.sig$logFC, pValue=tab.sig$P.Value, FDR=tab.sig$adj.P.Val, Ensembl=Ensembl)
```
Finally, we are now going to create one **html** file and one **text** file with the list of DEGs. The html file contains clickable links to the Ensembl Genome Browser.

```{r}
> HTML(tmp, "out.html", append=F)


## Week 6 Homework: :house: 

In this tutorial, we detected the DEGs between WT Control (3 samples) and KO Treatment at 24 hours (3 samples). Using similar analysis workflow, develop an R analysis pipleine (script) to detect the DEGs between WT Control and KO Treatment at 8 hour. Compare the list of top 20 DEGs between KO Treatment at 24 hours and KO Treatment at 8 hour.

Submit your homework via e-mail.










