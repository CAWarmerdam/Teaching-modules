---
title: "GWAS tutorial"
author: "Adriaan van der Graaf, Annique Claringbould"
date: "September 2, 2019"
output: 
  html_document:
    toc: TRUE
    number_sections: FALSE
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

<br><br><br>

## Introduction

This is a tutorial of a genome-wide association study (GWAS).

In essence, a genome-wide association study is an examination of a genome-wide set of variants in different individuals to see if any variant is associated with a phenotype. 

A GWAS can answer questions such as:

 * Which genetic variants are associated to Human height?
 * Are the genetics of the HLA region involved in celiac disease?
 * How much genetic risk does an individual have for a disease? (not in this tutorial)

A GWAS study investigates a number of individuals that have the following information available:

 * genotype information on a genome-wide scale
 * phenotype information 
 
Phenotype data is any data related to the individual: questionnaires, compounds measured in blood (such as LDL cholesterol), or if someone has a disease or not.
By using this information, a GWAS compares the DNA of individuals having a certain phenotype. For instance, some individuals have a disease (cases) and similar individuals may not have the disease (controls).
If one type of a genetic variant (one allele) is more frequent in cases, then the variant is identified as being associated with the disease. 
The associated SNPs are then considered to mark a region of the human genome that may influence the risk of developing a disease.

This tutorial provides a workflow of doing a GWAS on celiac disease. You will identify loci that are associated with increased risk to develop the disease. This information can be further used to investigate the molecular pathways that genes from the GWAS-identified loci are involved (a later tutorial). 

Celiac disease is an autoimmune disorder that can occur in genetically predisposed individuals and it affects 1 in 100 individuals worldwide. It is known that gluten consumption leads to damage in the small intestine, resulting in poor absorbency of nutrients from the human body. Celiac disease leads to growth defects in young children and can show up with a plethora of symptoms in adults.

This tutorial assumes you are familiar with R language, but it's not required to know each step, as answers will be provided to everyone. 
Furthermore, questions are provided to get familiar with every step of the GWAS workflow. 
Remember that doing associations is the same as doing statistics so a relatively small knowledge of maths is required. 
Please make an effort to understand the equations that are used here. 
Additionally all of the math used here is encoded in the R code that is provided by the tutorial.

Because of the sensitive nature of using individual's genetic data, *publicly* available genotypes and *simulated* phenotypes will be used in this tutorial. However, the workflow essentially remains the same if you use a data set that contains real phenotypes.

The GWAS data that will be generated for *Celiac disease* can be used for follow-up analysis in the next tutorials.

<br><br><br>

## Data used in this tutorial

The data in use is in the binary *plink* format, which consists of a BED, BIM and FAM file.

**BED file**
The BED file contains the genotypes called at bi-allelic variants, encoded in a binary format. A BED file must be accompanied by BIM and FAM files.

**BIM file**
The BIM file encodes information about each of the variable genotype locations, or SNPs. The fields in a BIM file are:
* Chromosome
* SNP marker ID
* Genetic distance
* Physical position
* Allele 1
* Allele 2

**FAM files**
The FAM file encodes information about each of the individuals that you are including in your analysis. The FAM file also includes the phenotype information. The fields in a FAM file are:
* Family ID (this column is used if you have multiple individuals from the same family)
* Individual ID
* Paternal ID (this column is used to indicate the Individual ID of the father, if you have family data)
* Maternal ID (this column is used to indicate the Individual ID of the mother, if you have family data)
* Sex (1 = male; 2 = female; other = unknown)
* Phenotype (1 = control; 2 = case; 0 = unknown)

Plink is a command-line program that allows a user to perform an integrated GWAS. It is a very fast and helpful program, but is not very transparent in its methods. For more information on plink, please use the link: https://www.cog-genomics.org/plink2. In this tutorial, we are doing a very similar analysis using R language. 

We will load the plink genotype and phenotype matrix using the R package `BEDMatrix`. The rest of the analysis will be done using base R. Some commands that are provided will be complicated (such as is the nature of R in some cases), but the comments in the code explain the steps that are taken.

Our  data consists of 512 individuals, and 131,118 bi-allelic genetic variants. Bi-allelic variants are variants where we only observe two alleles in the human population. The majority of SNPs are bi-allelic, but there are sites where three or even all four alleles occur. We restrict our analysis to bi-allelic variants because they are easy to analyze. 

<br><br><br>

## Downloading the data

You can download a zip file from [here](http://molgenis.org/downloads/gwas_tutorial_AvdG_2017/celiac_gwas.zip)

Make sure this zip file is unpacked in a unique folder. in our case this is the folder `genotypes`

Then point the r working directory to this folder using `setwd`.

<br>

#### Data conventions
If we want to do statistics on genotype data, these data need to be numerically encoded. In this tutorial, we choose to explain the *additive* effect of a certain allele.
For instance, take a hypothetical SNP `rs12` which has two alleles (remember the bi-allelic variants): `A` and `C`. The amount of individuals that have each of the possible genotypes of `rs12` (`AA`, `AC`, `CC`) are shown in the table below.<br>

```{r, message=FALSE, warning=FALSE, eval=T, echo=F}
a <- data.frame(genotype=c("AA","AC", "CC"), count=c(497,12,3))
a
```


<br>
The `A` allele is more prevalent so we call this the **major** allele, and the `C` allele is then the **minor** allele. We then encode the genotypes `AA`, `AC` and `CC` for each individual as $0, 1, 2$ respectively. This is the same as counting the number of `C` alleles in an individual.

Coding alleles numerically allows an investigator to do regression on the additive relationship of an allele and a phenotype.Dominant, recessive or epistatic effects are also present in biology, but we do not consider them in this tutorial.

<br><br><br>

## GWAS steps

The following steps are taken to perform a GWAS: 

 1. Import the data in R and explore them
 
 2. Quality control per SNP
      i) Remove SNPs with minor allele frequency (MAF) < 0.01
      i) Remove SNPs that deviate from Hardy Weinberg equilibrium (HWE) using a P threshold < 1e-6
      i) Remove SNPs with missing genotype rate < 0.99
 
 3. Quality control per individual
     i) Identify and remove related individuals
     i) Perform principal component analysis (PCA) and plot the first two components to identify and remove population outliers
 
 4. Perform genome-wide association testing
     i) do initial association analysis, linear
     i) do association analysis, logistic (Bonus)
     i) prepare a Manhattan plot


<br><br><br>

## Step 1. Import the data in R and explore them
<br>
Before we start, make sure you install the `BEDMatrix` package:
 
```
install.packages("BEDMatrix")
```

Now, download the data and set the R working directory to this directory as well (use `setwd`).
 
When the data is downloaded, let's have a look at it. Find the data in the folder 'genotypes'.

```{r, message=FALSE, warning=FALSE, eval=T}

require(BEDMatrix)
setwd("genotypes/")
#Import the data in R
genotype_matrix <- BEDMatrix("celiac_gwas.bed") #my data is stored in a directory called 'genotypes'
phenotypes <- read.table("celiac_gwas.fam", header=FALSE)[,6] #the sixth column of the fam file contains the phenotype matrix.
```

*Make sure this works, otherwise R is not in the working directory as where the data is stored.*

First, have a look at the phenotypes, plot a histogram of them (use the `hist()` function). In a GWAS, ideally the phenotypes should look normally distributed (i.e. it looks like a classic bell curve). If the phenotype data is not normally distributed, it is important to normalize it before running the association analysis.

Now if we look at the `genotype_matrix` object, we will see that it is a big matrix: each row represents one individual and each column a SNP.

```{r, message=FALSE, warning=FALSE, eval=FALSE, echo=FALSE}
dim(genotype_matrix) # how many variants and how many individuals are present

genotype_matrix[1:100,1] #shows 100 individual genotypes for the first SNP

colnames(genotype_matrix)[1:10] # shows the first 10 names of all SNPs
rownames(genotype_matrix)[1:10] # shows the first 10 names of all individuals in the dataset

genotype_matrix[1:3, 1:5] # shows the first three individuals and first five SNPs
#Note that SNP genotypes are encoded as 0 (AA), 1 (AB or BA) and 2 (BB)
```

**Q1**: How many SNPs and individuals are present in our data set before quality control?

**Q2**: Please plot the data of the 320th SNP with the phenotype (`plot(genotype_matrix[,320] ,phenotypes)`). Would you say this SNP is associated with the phenotype?

<br><br><br>

## Step 2: Quality control per SNP

The genotype data we have here is still fairly noisy. We will take three steps to ensure our genotypes are cleaned: 

1. Remove SNPs with low minor allele frequency (MAF < 0.01)
2. Remove SNPs that deviate from the Hardy-Weinberg equilibrium (HWE) using a P-value threshold < 1e-6
3. Remove SNPs with high missing genotype rate (missingness)

<br><br>

### Step 2.1: Remove SNPs with low minor allele frequency

Low minor allele frequency alleles can lead to false positive results in a GWAS. If the minor allele frequency is not determined properly, there is a chance that this leads to overfitting of the association in individuals with extreme phenotypes and alleles that are not present in the rest of the population.  
Because our data consists of 512 individuals, we choose a minor allele frequency threshold of 1 percent, which means that we expect to see the allele in total $0.01 * 512 * 2 = 10$ times in our study population. The multiplication by 2 is because every individual has 2 alleles.

The function for identifying the minor allele frequency per SNP is as follows:
```{r, message=FALSE, warning=FALSE, eval=TRUE, echo=TRUE}
#Count the number of minor alleles over all individuals and then divide it by 2*the number of individuals.
minor_allele_frequency <- function(row){
	a1_count = 2* sum(row == 2) + sum(row ==1)
	return(a1_count / (2*length(row)))
}
```
<br>
**Q3**: What is the MAF of the first SNP?

<br>
Now using the `apply()` function, we are able to get a vector of SNPs with minor allele frequencies called `minor_allele_frequency_per_snp`. We can then make a logical vector to identify SNPs with a MAF lower than 0.01 called `maf_filter`. 

```{r, message=FALSE, warning=FALSE, eval=TRUE, echo=FALSE}

#Now, do it for all the snps
minor_allele_frequency_per_snp <- apply(genotype_matrix, 2, minor_allele_frequency)

#Identify SNPs that have a MAF lower than 0.01. These SNPs will be removed in the next steps
maf_filter <- minor_allele_frequency_per_snp < 0.01 # logical vector

#Check how many SNPs show a MAF < 0.01 that will be removed
#sum(maf_filter, na.rm=T) #4098 SNPs
```
<br>
**Q4**: How many SNPs show a MAF < 0.01?

<br><br>

Now, we can remove the SNPs showing a MAF < 0.01:
```{r, message=FALSE, warning=FALSE, eval=TRUE}
genotype_matrix_maf_filtered <- genotype_matrix[,!maf_filter]
```

<br>
Make a histogram of the minor allele frequencies, to see if everything is correct before and after filtering. Use the `hist()` function.
```{r, message=FALSE, warning=FALSE, eval=TRUE, echo=FALSE}
hist(minor_allele_frequency_per_snp)
hist(minor_allele_frequency_per_snp[!maf_filter])
```

<br><br>
We will identify more genotypes that need to be filtered out in the next steps. Therefore, we will now keep the unfiltered genotype matrix, and only filter variants out when we known the total number of SNPs that should be removed in all the quality control steps.

<br><br>

### Step 2.2: Remove SNPs that deviate from HWE

The Hardy Weinberg equilibrium (HWE) is a principle stating that the genetic variation in a population will remain constant from one generation to the next in the absence of disturbing factors. Departure from this equilibrium can be an indicator of potential genotyping errors, population stratification or even actual association with the phenotype under study. In other words, the HWE states that the observed counts per genotype should match the following expectation, if there is random segregation of alleles: 
<br><br>
$$
Exp(AA) = (1-p)^2n\\
Exp(AB) = 2 p (1-p)n\\
Exp(BB) = p^2n
$$
<br>
Where $p$ is the minor allele frequency, $n$ is the number of individuals, $A$ represents the major allele and $B$ represents the minor allele. 
<br><br>
We can identify the expected counts for genotypes `AA`, `AB`, and `BB`, by plugging in the number of individuals and minor allele frequency for each SNP. Make a matrix with the expected genotype counts for `AA`, `AB`, `BB` using `cbind()`. 
<br><br>
The first 10 lines should look as follows:

```{r, message=FALSE, warning=FALSE, eval=TRUE, echo=FALSE}
#Check the number of individuals and SNPs before quality control
num_individuals <- nrow(genotype_matrix)
num_variants <- ncol(genotype_matrix)

#Find the expected genotype counts per SNP
#Make a matrix per column with the number of expected homozygote (AA), hetererozygote (AB) and homozygote (BB) genotypes per SNP
expected_genotype_count <- cbind( 
	(1-minor_allele_frequency_per_snp)^2 * num_individuals,
	2*(1-minor_allele_frequency_per_snp) * minor_allele_frequency_per_snp * num_individuals,
	minor_allele_frequency_per_snp^2 * num_individuals
	)

colnames(expected_genotype_count) <- c('exp_AA','exp_AB','exp_BB')

#Have a look at the first 10 SNPs
expected_genotype_count[1:10,]
```

<br>
We want to compare the expected with the observed values to see if there is a difference. We can use the Pearson chi-squared test to find alleles that are not in hardy Weinberg equilibrium (HWE). 
<br>
The Pearson $\chi^2$ test is the following:
<br><br>
$$
\chi^2 = \frac{(Obs(AA) - Exp(AA))^2}{Exp(AA)} + \frac{(Obs(AB) - Exp(AB))^2}{Exp(AB)} +   \frac{(Obs(BB) - Exp(BB))^2}{Exp(BB)}
$$
<br>
Where $Obs(AA)$ represents the observed number of homozygotes for the major allele, and $Exp(AA)$ represents the expected number of homozygotes for the same major allele.
  
Now, lets have a look at the observed counts of genotypes:

```{r, message=FALSE, warning=FALSE, eval=TRUE, echo=TRUE}
find_observed_count <- function(row){
	counts <- c(
			sum(row == 0, na.rm=T), #number of individuals with genotype AA
			sum(row == 1, na.rm=T), #number of individuals with genotype AB
			sum(row == 2, na.rm=T)  #number of individuals with genotype BB
		)
	return(counts)
}
```

<br>
**Q5**: How many homozygote (AA), heterozygote (AB) and homozygote (BB) genotypes for SNP 320 can you observe?
<br>
```{r, message=FALSE, warning=FALSE, eval=FALSE, echo=FALSE}
#Try this function for SNP 320
find_observed_count(genotype_matrix[,320])
```

<br>
Make a matrix of the observed counts using the function `find_observed_count()`, and `t(matrix(apply()))`. Hint: you will need to specify that the number of rows is 3 with `nrow=3`. The output should look like this:
```{r, message=FALSE, warning=FALSE, eval=TRUE, echo=FALSE}
#Calculate observed genotype counts for all SNPs
observed_genotype_count <- t(matrix(apply(genotype_matrix, 2, find_observed_count), nrow=3))

colnames(observed_genotype_count) <- c('obs_AA','obs_AB','obs_BB')

observed_genotype_count[1:10,]
```

<br>
Compare the observed with expected genotype counts and calculate the chi-squared statistics

```{r, message=FALSE, warning=FALSE, eval=TRUE }
#now compare observed with expected, and get the chi sq statistic.
chi_sq_statistics <- apply((observed_genotype_count - expected_genotype_count)^2 / expected_genotype_count, 1, function(x) sum(x, na.rm=T))
```

<br>
Explore the chi-squared statistics by plotting a histogram, and checking the minimum and the maximum values.

<br>
**Q6**: What is the range of chi-squared values? Does a high chi-squared value indicate that there is a big difference between observed and expected genotypes or not?

```{r, message=FALSE, warning=FALSE, eval=FALSE, echo=FALSE}
hist(chi_sq_statistics)
min(chi_sq_statistics)
max(chi_sq_statistics)
```

<br>
In order to identify the P-value that belongs these chi-squared valued, we can look up each of the chi-squared values in a standard chi-squared distribution with one degree of freedom. The idea here is that you compare the chi-squared values we have just calculated with the expected chi-squared values to see if they deviate very much.
We can find the P-values as follows:
```{r, message=FALSE, warning=FALSE, eval=TRUE }
hardy_weinberg_p_values <- 1 - pchisq(chi_sq_statistics,1)
```

<br>
Make a logical vector in the same way that you did for the minor allele frequency filter. This vector contains TRUE/FALSE values for each SNP indicating if it deviates from HWE (P < 1e-6), which we can use to filter out the SNPs at a later step.

```{r, message=FALSE, warning=FALSE, eval=TRUE, echo=FALSE}
hwe_p_val_filter <- hardy_weinberg_p_values < 1e-6
```

**Q7**: How many SNPs deviate from HWE?
```{r, message=FALSE, warning=FALSE, eval=FALSE, echo=FALSE}
# Check how many SNPs deviate from HWE and should be removed later
genotype_matrix_hwe_filtered <- genotype_matrix[,!hwe_p_val_filter]
dim(genotype_matrix_hwe_filtered)
```

<br><br>

### Step 2.3: Remove SNPs with high missing genotype rate
This step is relatively simple, but it is required for most analyses. 
To make sure that there are no SNPs that are missing due to technical biases, we remove all the SNPs that have at least one value missing. Missing values are encoded as `NA` in the genotype_matrix. We can identify a missing value by using the `is.na()` function in R. We can calculate the missingness per SNP by summing the number of `NA` values for one SNP across all individuals, and dividing over the number of individuals. Use `apply()` and `sum(is.na())` to calculate a vector of missingness.

```{r, message=FALSE, warning=FALSE, eval=TRUE, echo=FALSE}
#missing snps are encoded as NA in our data.  so we count them and divide over the number of individuals.
missingness <- apply(genotype_matrix, 2, function(x) sum(is.na(x))) / num_individuals
```

<br>
Plot the distribution of missingness using `hist()`.
```{r, message=FALSE, warning=FALSE, eval=TRUE, echo=FALSE}
#missing snps are encoded as NA in our data.  so we count them and divide over the number of individuals.
hist(missingness)
```

<br><br>
Based on these results, genotyping seems to be done well: the missingness is either 0 (no missingness at all) or almost 1 (SNPs are missing across all individuals). Create a logical vector, like we did before, with SNPs that have `missingness != 0`.
```{r, message=FALSE, warning=FALSE, eval=TRUE, echo=FALSE}
missingness_filter <- missingness != 0
```

**Q8**: How many SNPs show high genotype rate missingness?
``` {r, message=FALSE, warning=FALSE, eval=FALSE, echo=FALSE}
genotype_matrix_missingness_filtered <- genotype_matrix[,!missingness_filter]
dim(genotype_matrix_missingness_filtered)
```

<br><br>

### Step 2.4: Filter out SNPs

We have just identified SNPs that should be filtered out based on one (or more) of these criteria:
* MAF < 0.01
* HWE P < 1e-6
* high missingness (encoded as NA)

We would like to keep only the genotypes for the SNPs that pass these criteria. We also want to keep only the MAF measurements for those genotypes:

```{r, message=FALSE, warning=FALSE, eval=TRUE, echo=TRUE }
# We can now filter out all snps, and save the filtered matrix
genotype_matrix_geno_qc <- genotype_matrix[,!missingness_filter & !hwe_p_val_filter & !maf_filter]
minor_allele_frequency_geno_qc <- minor_allele_frequency_per_snp[!missingness_filter & !hwe_p_val_filter & !maf_filter]
```

```{r, message=FALSE, warning=FALSE, eval=FALSE, echo=FALSE }
#Check how many SNPs and individuals remain after quality control
dim(genotype_matrix_geno_qc)
```

<br>
**Q9**: After quality control per SNP, how many SNPs have been filtered out?
 
<br><br><br>

## Step 3: Quality control per individual 
We have filtered the genotypes and we can now proceed to filtering out individuals. This analysis excludes individuals based on two criteria: 
* relatedness 
* population outliers
<br><br>
A relatedness check (where you test if individuals in your cohort are genetically related) can potentially help identify duplicated and/or contaminated samples.  
Removing population outliers is critical in genetic studies. Population stratification (the presence of a systematic difference in allele frequencies between subpopulations) is a main source of confounding that can lead to spurious associations. Population stratification can create genotypic differences between cases and controls that are not due to their difference in disease status (our goal) but due to different population origins in cases vs. controls. To be sure we do not pick up those effects, we will remove population outliers.

<br><br>

### Step 3.1: Identify related individuals
To identify related individuals, we make a genetic relationship matrix (GRM). This matrix identifies how related the individuals are by comparing all genotypes of the individuals. The more similar are two individuals on the DNA level, the more likely they are to be related. For example, two siblings share on average 50% of genetic variants, while a grandparent-grandchild pair shares ~25%. In our data we should have unrelated individuals (no families), so relatedness in our data might indicate contamination or duplication of the samples.

<br>
The GRM can be calculated as follows:

```{r, message=FALSE, warning=FALSE, eval=TRUE }

#Make a genetic relatedness matrix (complicated in the math)

maf_normalizer <- sqrt(2*(1-minor_allele_frequency_geno_qc)*minor_allele_frequency_geno_qc)

#Following Yang et al 2010:
# grm = W %*% W / N
# where w_{ij} = (x_{ij} - 2*p) / sqrt(2*p*(1-p))
# p is the allele frequency.

w_mat <- t(apply(genotype_matrix_geno_qc, 
				1,
			 	function(x) as.vector((x - 2*(minor_allele_frequency_geno_qc)) / maf_normalizer)
			 	)
			)

# Takes about a minute but 2 GB of memory
grm <- (w_mat %*% t(w_mat)) / num_variants

```

<br>
Create a heat map to check the relatedness of all individuals. Use the function `heatmap()` with the `scale = "none"` option. Can you recognise any potential contaminations?
```{r, message=FALSE, warning=FALSE, eval=FALSE, echo=FALSE}
Colors=colorRampPalette(c('blue','white','red'))(20)
Breaks = c(seq(-1,1,by=0.1))

heatmap(grm, col = Colors, breaks = Breaks, Rowv = NA, Colv = NA, scale = "none")
```

<br>
Plot a histogram of the relatedness using `hist()`. The relatedness information is stored in the lower triangle of the matrix: `grm[lower.tri(grm)]`.
```{r, message=FALSE, warning=FALSE, eval=FALSE, echo=FALSE }
hist(grm[lower.tri(grm)])
```

<br>
We set a cutoff of 0.1 for relatedness, which roughly means that we exclude individuals that are more related to each other than you would be related to a great great grandparent. Remove the individuals that are more than 0.1 related to each other.

```{r, message=FALSE, warning=FALSE, eval=TRUE }

# Remove individuals that show a relatedness value more than 0.1 
relatedness_indice <- which(grm > 0.1, arr.ind=TRUE) #find the indices in the matrix.

relatednes_individuals <- unique(relatedness_indice[relatedness_indice[,1] != relatedness_indice[,2],][,1]) #make sure not to remove individuals which are compared as the same.

# For simplicity's sake, we remove all related individuals.
# and apply the relatedness filter.
relatedness_filter <- rep(FALSE, num_individuals)
relatedness_filter[relatednes_individuals] <- TRUE
```

<br>
**Q10**: How many individuals should be excluded due to relatedness?
```{r, message=FALSE, warning=FALSE, eval=FALSE, echo=FALSE}

#Check the number of related individuals
genotype_matrix_relatedness_filtered <- genotype_matrix_geno_qc[!relatedness_filter,]
dim(genotype_matrix_geno_qc)[1] - dim(genotype_matrix_relatedness_filtered)[1]
```

<br><br>

### Step 3.2: Identify population outliers
To identify population outliers, we perform a principal components analysis (PCA). PCA is a procedure that summarizes information into new variables called principal components (PCs). PCs are linearly uncorrelated, so that means that each PC captures a different part of the genotype information. The first PC captures most variation, the second PC captures the second most variation, etc. Because population structure has a very strong genetic signal, the first few PCs almost always capture population differences. For a striking example of European population stratification, see Novembre et al., 2008, Nature (https://www.nature.com/articles/nature07331.pdf).

<br>
Calculate the PCs by using the function `prcomp()` on the GRM. Save them as `principal_components`.
```{r, message=FALSE, warning=FALSE, eval=TRUE, echo=FALSE}
principal_components <- prcomp(grm) # relatively easy in R.
```

<br>
Plot the first two PCs against each other using `plot()`. The values of the third PC can be accessed as `principal_components$x[,3]`.
```{r, message=FALSE, warning=FALSE, eval=FALSE, echo=FALSE}
plot(principal_components$x[,1], principal_components$x[,2]) # plot the first two principal components
```

<br>
You will now filter out the outlying individuals using a PC threshold that you define based on the plot. 
<br>

**Q11**: Are there any clear clusters or outliers? What threshold would you propose?

<br>
Choose a threshold that eliminates all clear outliers. Make a logical vector `pca_filter` with TRUE/FALSE values for each individual based on whether they are an outlier given your threshold. A table of this logical vector should look like this:
```{r, message=FALSE, warning=FALSE, eval=TRUE, echo=FALSE}
#We decide on a threhold for the principal components to filter on

pca_filter <- principal_components$x[,2] < -0.4
table(pca_filter)
```

<br>
Identify the individuals that do not pass the PCA filter:
```{r, message=FALSE, warning=FALSE, eval=FALSE, echo=TRUE}
#Identify the individuals that show PCAs < threshold
rownames(genotype_matrix_geno_qc)[pca_filter]

#Check how many individuals are identified as population outliers
genotype_matrix_pcas_filtered <- genotype_matrix_geno_qc[!pca_filter,]
dim(genotype_matrix_pcas_filtered)
```

<br>
Remove individuals that do not pass the pca_filter or relatedness_filter from the `genotype_matrix_geno_qc` and from the `phenotypes` matrices. Call the new matrices `genotype_matrix_post_qc` and `phenotypes_post_qc`.
```{r, message=FALSE, warning=FALSE, eval=TRUE, echo=FALSE}
#Remove individuals from the genotype matrix based on relatedness and PCA, and we have concluded our QC steps
genotype_matrix_post_qc <- genotype_matrix_geno_qc[!pca_filter & !relatedness_filter,]

#Also, remove these individuals from the phenotypes.
phenotypes_post_qc <- phenotypes[!pca_filter & !relatedness_filter]
```

<br>
**Q12**: How many individuals remain after quality control on the individuals?

```{r, message=FALSE, warning=FALSE, eval=FALSE, echo=FALSE}
dim(genotype_matrix_post_qc) # leaves 500 individuals! 114404 variants!
```
<br><br><br>

## Step 4: Testing for genetic associations

After all these steps of quality control, it is time to get started with the actual association analysis! Let's test whether our genetic variants are associated with celiac disease.
<br><br>
Using a linear model, we can associate all the genotypes to the disease. We can use the function `lm()` to test whether one SNP is associated with celiac disease like this:
```{r, message=FALSE, warning=FALSE, eval=TRUE, echo=TRUE}
summary(lm(genotype_matrix_post_qc[,1]~phenotypes_post_qc))
```

<br>
Next, we create an R function to get association statistics for all genotypes. We are mostly interested in the effect size of the association, and its P-value. This information in present in the second line of Coefficient information in the summary above. The function saves for each SNP the slope estimate, standard error, t-value and p-value:
```{r, message=FALSE, warning=FALSE, eval=TRUE, echo=TRUE}
do_quantative_association <- function(genotypes, phenotypes){

	sumdat <- summary(lm(genotypes~phenotypes))
	return(as.vector(sumdat$coefficients[2,])) #second row is the column.
}
```

<br>
Apply the model to all genotypes using `t(matrix(apply())`, be sure to include the `phenotypes_post_qc` as the second argument, and indicate that `nrow=4`. Save the output as `associations`. This analysis should take a little while (~2 min). The resulting `associations` should look like this:
```{r, message=FALSE, warning=FALSE, eval=TRUE, echo=FALSE}

# It takes a while (about 2 minutes)
associations <- t(matrix(apply(genotype_matrix_post_qc, 2, do_quantative_association, phenotypes=phenotypes_post_qc), nrow=4))

head(associations)
```

<br>
We have calculated the associations and we would like to plot the significance level over the chromosomal positions (*Manhattan plot*).
SNP names are in the format `<chr>:<position>_<effect_allele>`. Therefore, we need to split this format on `:` and on `_` and turn it into a dataframe.

```{r, message=FALSE, warning=FALSE, eval=TRUE, echo=TRUE}

positions_of_snps <- do.call(rbind,strsplit(colnames(genotype_matrix_post_qc), ":|_"))

associations_with_position <- cbind(positions_of_snps, associations)

#Make a dataframe for plotting with ffplot
associations_with_position_df <- data.frame(chr = as.numeric(associations_with_position[,1]), 
                                 position = as.numeric(associations_with_position[,2]),
                                 allele = associations_with_position[,3],
                                 beta = as.numeric(associations_with_position[,4]),
                                 se = as.numeric(associations_with_position[,5]),
                                 t_stat = as.numeric(associations_with_position[,6]),
                                 p_val = as.numeric(associations_with_position[,7]))
```

<br>
Finally, we plot the Manhattan plot using the package `ggplot2`. We want the x-axis to reflect the chromosomal `position` and the y-axis should be the `-log10(p_val)`. It's good practice to distinguish the chromosomes by giving each a different colour: `col=as.factor(chr))`. Lastly, since we only know the positions of each SNP per chromosome, we want to add `facet_grid(.~chr, scales="free_x")`. If everything worked, it should look like this:

```{r, message=FALSE, warning=FALSE, eval=TRUE, echo=FALSE}
require(ggplot2)
ggplot(associations_with_position_df, aes(x=position, y=-log10(p_val), col=as.factor(chr))) + 
  facet_grid(.~chr, scales="free_x") + 
  geom_point()
```

<br>
The standard genome wide significance threshold is thought to be P < 5E-8. Add this significance line to the plot using `geom_hline()`. Remember the scale of the y-axis when you add the threshold!
```{r, message=FALSE, warning=FALSE, eval=FALSE, echo=FALSE}
pval <- 5e-8
require(ggplot2)
ggplot(associations_with_position_df, aes(x=position, y=-log10(p_val), col=as.factor(chr))) + 
  facet_grid(.~chr, scales="free_x") + 
  geom_point() +
  geom_hline(yintercept = -log10(pval))
```

<br>
Save the plot as a PNG file using `ggsave("assocplot.png", width=8, height=4.5, dpi=300)`.

<br>
**Q13**: What chromosomes harbour significant signals if you look at the plot?

<br>
**Q14**: How many SNPs in total pass the significance threshold for GWAS?
```{r, message=FALSE, warning=FALSE, eval=FALSE, echo=FALSE}
sign_assoc <- associations_with_position_df[associations_with_position_df$p_val < pval,]
dim(sign_assoc)[1]
```

<br>
Last command:
```{r, message=FALSE, warning=FALSE, eval=FALSE, echo=TRUE}
install.packages('praise')
praise()
```

<br>
You are now done! Please answer 3 questions about today here: https://bit.ly/2Pm7hxf

<br><br><br><br><br><br><br><br>
