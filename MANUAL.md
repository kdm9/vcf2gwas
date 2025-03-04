# *vcf2gwas* manual

## About

This manual is meant to clarify the most important options and possibilities when running *vcf2gwas*.  
**Note**: When running *vcf2gwas* via [docker](https://www.docker.com/), replace `vcf2gwas` in every command with `docker run -v /path/to/current-working-directory/:/vcf2gwas/ fvogt257/vcf2gwas`.  
To obtain example files for running *vcf2gwas*, run:

```
vcf2gwas -v test
```

Besides testing the installation, this will copy the example VCF file `example.vcf.gz` and phenotype file `example.csv` to your current working directory.  
The VCF file `example.vcf.gz` contains pre-filtered SNP information of about 60 accessions of *A. thaliana* from the [1001 Genomes](https://1001genomes.org/accessions.html) project.  
The phenotype file `example.csv` is made up of the `avrRpm1` phenotype from the [AraPheno](https://arapheno.1001genomes.org/phenotype/17/) database for the same accessions.

## Contents

* [GEMMA models](#gemma-models)
    * [Running a linear model analysis](#running-a-linear-model-analysis)
    * [Running a linear mixed model analysis](#running-a-linear-mixed-model-analysis)
    * [Running a bayesian sparse linear mixed model analysis](#running-a-bayesian-sparse-linear-mixed-model-analysis)
* [File related options](#file-related-options)
    * [Selecting multiple phenotype files](#selecting-multiple-phenotype-files)
    * [Analyzing multiple phenotypes](#analyzing-multiple-phenotypes)
    * [Transforming phenotype values](#transforming-phenotype-values)
    * [Adding covariates](#adding-covariates)
    * [Comparing results to specific genes](#comparing-results-to-specific-genes)
    * [Adding a relatedness matrix](#adding-a-relatedness-matrix)
    * [Analyzing a subset of chromosomes](#analyzing-a-subset-of-chromosomes)
    * [Changing the output directory](#changing-the-output-directory)
* [Miscellaneous options](#miscellaneous-options)
    * [Limiting memory and core usage](#limiting-memory-and-core-usage)
    * [Using dimensionality reduction of phenotypes for analysis](#using-dimensionality-reduction-of-phenotypes-for-analysis)
    * [Using PCA of genotypes instead of standard relatedness matrix](#using-pca-of-genotypes-instead-of-standard-relatedness-matrix)
    * [Filter out SNPs](#filter-out-snps)
    * [Change manhattan plot](#change-manhattan-plot)
    * [Change font size in all plots](#change-font-size-in-all-plots)
    * [Keep temporary files](#keep-temporary-files)
* [Output](#output)
    * [Quality Control](#quality-control)
    * [Plots](#plots)
    * [Summaries](#summaries)

## GEMMA models

This sections demonstrates the usage of *vcf2gwas* when running the different analysis models of GEMMA.  

To read more in detail about the different models GEMMA is able to run, please refer to their [manual](https://github.com/genetics-statistics/GEMMA/blob/master/doc/manual.pdf).

### Running a linear model analysis

To run a linear model analysis with default settings, select a `VCF` file with the `-v/--vcf` and a phenotype file with the `-pf/--pfile` option, respectively. To select a specific phenotype from the phenotype file, utilize the `-p/--pheno` option:

```
vcf2gwas -v [filename] -pf [filename] -p 1 -lm
```

This will use the first phenotype column in the phenotype file for analysis.

### Running a linear mixed model analysis

GEMMA offers univariate and multivariate linear mixed model analysis. 

#### Univariate linear mixed model

To run a linear mixed model analysis with default settings, type:

```
vcf2gwas -v [filename] -pf [filename] -p 1 -lmm
```

This again uses the first phenotype column in the phenotype file to carry out the analysis.  
To analyse multiple phenotypes (for example the 1st, 2nd and 4th) independently, run:

```
vcf2gwas -v [filename] -pf [filename] -p 1 -p 2 -p 4 -lmm
```

This will carry out the univariate linear mixed model analysis for the 1st, 2nd and 4th phenotype column in the phenotype file.

#### Multivariate linear mixed model

To analyse these phenotypes jointly, employ the `-m/--multi` option:

```
vcf2gwas -v [filename] -pf [filename] -p 1 -p 2 -p 4 -lmm --multi
```

### Running a bayesian sparse linear mixed model analysis

Run a bayesian sparse linear mixed model analysis with any phenotype in the phenotype file:

```
vcf2gwas -v [filename] -pf [filename] -p [int] -bslmm
```

To change to amount of burn-in steps, sampling steps and maximum value for 'gamma' from their default values (100,000, 1,000,000 and 300, respectively), use the `-w/--burn`, `-s/--sampling` and `-smax/--snpmax` options, respectively.

## File related options

*vcf2gwas* is capable of analyzing multiple phenotypes from one or multiple phenotypes at once. 
Depending on the available cores and memory of the machine running the analysis and the number of phenotypes to be analyzed, the phenotype file will be split up during the analysis to ensure maximum efficiency by analyzing phenotypes in parallel.

### Selecting multiple phenotype files

*vcf2gwas* is able to take in multiple phenotype files by employing the `-pf/--pfile` option for each file:

```
vcf2gwas -v [filename] -pf [filename1] -pf [filename2] -p [int] -lmm
```

If multiple phenotypes are specified, these phenotypes will be analyzed in every phenotype file.
This is advantageous if one wants to analyze the same phenotypes with a different subset of the individuals in the `VCF` file. 

### Analyzing multiple phenotypes

To analyze multiple phenotypes in one run you can either specify multiple phenotypes (by either typing the phenotype names or specifying the phenotype column position)

```
vcf2gwas -v [filename] -pf [filename] -p [name1] -p [name2] -p [name3] -lmm

vcf2gwas -v [filename] -pf [filename] -p [num1] -p [num2] -p [num3] -lmm
```

or select all phenotypes in the phenotype file at once utilizing the `-ap/--allphenotypes` option:

```
vcf2gwas -v [filename] -pf [filename] -ap -lmm
```

### Transforming phenotype values

vcf2gwas offers the option to transform the phenotype values in the phenotype file(s). The selected metric (default: 'wisconsin') is applied across rows.  
To transform the phenotypes, employ the `-t/--transform` option and to change the metric to one of the other supported metrics, add it as an argument to the option:

```
vcf2gwas -v [filename] -pf [filename1] -p [int] -lmm -t hellinger
```

Now, vcf2gwas, transforms the phenotypes according to the hellinger metric.  
The following metrics are available:
* *total*: Divides each observation by row sum
* *max*: Divides each observation by row max
* *normalize*: Chord transformation, also euclidean normalization, making the length of each row 1
* *range*: Converts the range of the data to 0 and 1
* *standardize*: Standardizes each observation (i.e. z-score)
* *hellinger*: Square-root of the total transformation
* *log*: Returns ln(x+1)
* *logp1*: Returns ln(x) + 1, if x > 0. Otherwise returns 0
* *pa*: Converts data to binary absence (0) presence (1) data
* *wisconsin*: First divides an observation by the max of the column, then the sum of the row. That is, it applies ‘max’ down columns then ‘total’ across rows

These functions are taken from the [ecopy](https://ecopy.readthedocs.io/en/latest/index.html) package.

**Note**: If desired, one can also use the [dimensionality reduction](#using-dimensionality-reduction-of-phenotypes-for-analysis) options in conjunction with the transformation. vcf2gwas will first transform the phenotypes and then reduce the dimensionality of the transformed phenotypes according to the chosen method and use these results as phenotypes.

### Adding covariates

GEMMA supports adding covariates to the linear model and the linear mixed model analysis.  
To extract principal components from the `VCF` file for subsequent use as covariates in the analysis, use `-cf/--cfile` and `-c/--covar` with the 'PCA' argument and the amount of PCs desired for the analysis, respectively:

```
vcf2gwas -v [filename] -pf [filename] -p [num1] -cf PCA -c 2 -lmm
```

Now, 2 principal components will be extracted from the `VCF` file and used for the linear mixed model analysis.  

To use principal components or UMAP embeddings of the phenotypic data as covariates for the analysis, see [Using dimensionality reduction of phenotypes for analysis](#using-dimensionality-reduction-of-phenotypes-for-analysis).  

Alternatively, a covariate file formatted in the same way as the phenotype file can be added manually.
Selecting covariates from a covariate file follows the same scheme as selecting phenotypes by using `-cf/--cfile` and `-c/--covar`:

```
vcf2gwas -v [filename] -pf [filename] -p [num1] -cf [filename] -c 1 -lmm
```

Here, the 1st covariate column of the covariate file will be considered in the analysis of the selected phenotype.
Similarly to the phenotype options, multiple covariates can be selected (either by name or column number) as well as all at once using `-ac/--allcovariates`.  
**Note**: Even if no covariates were added to the analysis, GEMMA will always use the intercept as a covariate.

### Comparing results to specific genes

Comparing the GWAS results to specific genes of interest can be a tedious task. To facilitate this process, *vcf2gwas* comes with GFF gene files already built-in for the most common species. Comparison to a species can be selected by employing the `-gf/--genefile` option and the abbreviation for the species:

```
vcf2gwas -v [filename] -pf [filename] -p 1 -lmm -gf [abbreviation]
```

Below are all supported species, their abbreviations as well as the used references and their sources:

|Species|Abbreviation|Scientific name|Reference|Source|
|---|---|---|---|---|
|Anopheles|AG|anopheles gambiae|AgamP4.51|[link](http://ftp.ensemblgenomes.org/pub/metazoa/release-51/gff3/anopheles_gambiae/)|
|Arabidopsis|AT|arabidopsis thaliana|TAIR10.51|[link](http://ftp.ensemblgenomes.org/pub/plants/release-51/gff3/arabidopsis_thaliana/)|
|C. elegans|CE|caenorhabditis elegans|WBcel235|[link](http://ftp.ensembl.org/pub/release-104/gff3/caenorhabditis_elegans/)|
|Fruit fly|DM|drosophila melanogaster|BDGP6.32|[link](http://ftp.ensembl.org/pub/release-104/gff3/drosophila_melanogaster/)|
|Zebrafish|DR|danio rerio|GRCz11|[link](http://ftp.ensembl.org/pub/release-104/gff3/danio_rerio/)|
|Chicken|GG|gallus gallus|GRCg6a|[link](http://ftp.ensembl.org/pub/release-104/gff3/gallus_gallus/)|
|Human|HS|homo sapiens|GRCh38.p13|[link](http://ftp.ensembl.org/pub/release-104/gff3/homo_sapiens/)|
|Mouse|MM|mus musculus|GRCm39|[link](http://ftp.ensembl.org/pub/release-104/gff3/mus_musculus/)|
|Rice|OS|oryza sativa|IRGSP-1.0.51|[link](http://ftp.ensemblgenomes.org/pub/plants/release-51/gff3/oryza_sativa/)|
|Rat|RN|rattus norvegicus|Rnor_6.0|[link](http://ftp.ensembl.org/pub/release-104/gff3/rattus_norvegicus/)|
|Yeast|SC|saccharomyces cerevisiae|R64-1-1|[link](http://ftp.ensembl.org/pub/release-104/gff3/saccharomyces_cerevisiae/)|
|Tomato|SL|solanum lycopersicum|SL3.0.51|[link](http://ftp.ensemblgenomes.org/pub/plants/release-51/gff3/solanum_lycopersicum/)|
|Grape|VV|vitis vinifera|12X.51|[link](http://ftp.ensemblgenomes.org/pub/plants/release-51/gff3/vitis_vinifera/)|
|Maize|ZM|zea mays|Zm-B73-REFERENCE-NAM-5.0.51|[link](http://ftp.ensemblgenomes.org/pub/plants/release-51/gff3/zea_mays/)|

Futhermore, *vcf2gwas* supports adding a 'gene-file' (either a GFF3 formatted `.gff` or comma-separated `.csv` file) containing the position as well as additional information of genes to the analysis by using the `-gf/--genefile` option:

```
vcf2gwas -v [filename] -pf [filename] -p 1 -lmm -gf [filename]
```

One can also add multiple 'gene-files' or use the built-in files in combination with 'gene-files' to efficiently compare the results of the analysis with different subsets of genes.

```
vcf2gwas -v [filename] -pf [filename] -p 1 -lmm -gf [abbreviation] -gf [filename1] -gf [filename2]
```

If the file is in the `.csv` format, the file needs at least three columns containing information about chromosome, gene start position and gene stop position. These columns have to be named 'chr', 'start' and 'stop'.  

*vcf2gwas* recognizes chromosomes in the following formats (here the first chromosome): `Chr1`, `chr1`, `1`.  
If the chromosomes in the `VCF` file are of a different format, it is necessary that the chromosome information in the gene file is formatted in the same way, otherwise *vcf2gwas* won't recognize the information correctly.  

*vcf2gwas* will summarize the n best SNPs (specified with `-ts/--topsnp`) of every analyzed phenotype and compare them to the genes in the file by calculating the distance between each SNP and gene upstream as well as downstream. These results can be filtered by saving only those SNPs with a distance to a gene lower than a specific threshold (set with `-gt/--genethresh`).  

**Note**: Since for each SNP only the gene with the closest start/end upstream and downstream is shown, this feature only serves to give a hint of possibly associated genes. Closer inspection by the user is strongly recommended.

### Adding a relatedness matrix

Although *vcf2gwas* will by default calculate a relatedness matrix depending on the chosen model, one may want to add a different one instead to the analysis. This is possible by employing the `-k/--relmatrix` option:

```
vcf2gwas -v [filename] -pf [filename] -p 1 -lmm -k [filename]
```

To use *vcf2gwas* to just calculate a relatedness matrix from the VCF file, run the `-gk` option:

```
vcf2gwas -v [filename] -gk 
```

To calculate the relatedness matrix and perform its eigen-decomposition in the same run, use the `-eigen` option:

```
vcf2gwas -v [filename] -eigen
```

Of course the `-eigen` option can also be used when supplying your own relatedness matrix with the `-k/--relmatrix` option.

### Analyzing a subset of chromosomes

By default *vcf2gwas* analyzes all chromosomes available in the `VCF` file. If one wants to analyze only a subset of the chromosomes, employ the `-chr/--chromosome` option:

```
vcf2gwas -v [filename] -pf [filename] -p 1 -lmm -chr [num1] -chr [num2]
```

Here, only the two specified chromosomes chromosomes will be used for the analysis, speeding up the whole process.

### Changing the output directory

By default, *vcf2gwas* will save the output in the current working directory. To change to a unique output directory, use the `-o/--output` option to specify a path:

```
vcf2gwas -v [filename] -pf [filename] -p 1 -lmm -o dir/example/
```

## Miscellaneous options

In this section other useful options of *vcf2gwas* will be elucidated.

### Limiting memory and core usage

By default *vcf2gwas* will use half of the available memory and all logical cores minus one. It can be important to limit usage of these resources especially when running the analysis on a machine shared with others.  
To set the memory (in MB) and core usage employ the `-M/--memory` and `-T/--threads` option, respectively:

```
vcf2gwas -v [filename] -pf [filename] -p 1 -lmm -M 8000 -T 6
```

Now, *vcf2gwas* uses 8 GB of memory and 6 cores to carry out the analysis.  
It is recommended to not set the memory to less than 1 GB.

### Using dimensionality reduction of phenotypes for analysis

When analyzing many phenotypes it can be escpecially beneficial to reduce the phenotypic dimensions. This allows the user to analyze or account for underlying structures in their phenotypic data by either using the output of the dimensionality reduction as phenotypes for GEMMA or adding them as covariates to the analysis.  
*vcf2gwas* offers two often-used methods to reduce the dimensions: principal component analysis ([PCA](https://en.wikipedia.org/wiki/Principal_component_analysis)) and Uniform Manifold Approximation and Projection ([UMAP](https://arxiv.org/abs/1802.03426)).  
When using the ouput as phenotypes, both methods can be used either separately or simultaneously in the analysis.

#### PCA

To perform PCA on the phenotype data and use the principal components as phenotypes for the analysis, use the `-P/--PCA` option.  
By default, *vcf2gwas* will reduce the phenotype dimensionality to 2 PCs. To change this value to any value between 2 and 10, append the value to the option:

```
vcf2gwas -v [filename] -pf [filename] -p 1 -lmm -P 3
```

Now, *vcf2gwas* will reduce the phenotype dimensionality to 3 instead of 2. *vcf2gwas* will also plot the variance explained of the principal components, so that the user can estimate the amount of sufficient PCs.

#### UMAP

To perform UMAP reduction on the phenotype data and use the embeddings as phenotypes for the analysis, use the `-U/--UMAP` option.  
By default, *vcf2gwas* will reduce the phenotype dimensionality to 2 embeddings. To change this value to any value between 1 and 5, append the value to the option:

```
vcf2gwas -v [filename] -pf [filename] -p 1 -lmm -U 3
```

Now, *vcf2gwas* will reduce the phenotype dimensionality to 3 instead of 2.  

By default, *vcf2gwas* uses euclidean metric for UMAP calculations. The metric can be changed with the `-um/--umapmetric` option:

```
vcf2gwas -v [filename] -pf [filename] -p 1 -lmm -U 3 -um manhattan
```

The manhattan metric is now used to calculate the UMAP embeddings.  
The following is a list of the available metrics:  
* *euclidean*
* *manhattan*
* *braycurtis*
* *cosine*
* *hamming*
* *jaccard*
* *hellinger*

#### Using PCs or UMAP embeddings as covariates

Another useful option is `-asc/--ascovariates`. It allows the user to utilize a variable amount principal components or UMAP embeddings of their phenotypic data as covariates in the analysis of said phenotypes:

```
vcf2gwas -v [filename] -pf [filename] -p 1 -lmm -U 3 -asc
```

Here, the phenotypic dimensionality will be reduced to 3 UMAP embeddings which will be subsequently used as covariates for the analysis.  
**Note**: This option takes only effect in conjuction with either the `-U/--UMAP` or `-P/--PCA` option, but not both simultaneously. 
Furthermore, only one phenotype file can be added to the analysis to take advantage of this option.

### Using PCA of genotypes instead of standard relatedness matrix

*vcf2gwas* uses GEMMAs standard method of kinship calculation for the linear mixed model, which produces a relatedness matrix. Instead of using this standard method, the relatedness matrix can optionally be calculated via PCA by utilizing the `-KC/--kcpca` option.  
The SNP data from the VCF file will be pruned by linkage disequilibrium with a default r-squared threshold of 0.5. To change the threshold, append the value to the option:

```
vcf2gwas -v [filename] -pf [filename] -p 1 -lmm -KC 0.8
```

### Filter out SNPs

By default, *vcf2gwas* will filter out SNPs with a minimum allele frequency of 0.01. To change this threshold use the `-q/--minaf` option:

```
vcf2gwas -v [filename] -pf [filename] -p 1 -lmm -q 0.02
```

### Change manhattan plot

The manhattan plot which will be produced from the GEMMA output, features by default a significant value line. The significance level is determined by Bonferroni-correcting the standard significance level of 0.05 with the amount of SNPs used by GEMMA in the analysis. All SNPs above that line will be labeled.  
To change the threshold line manually, use the `-sv/--sigval` option.

```
vcf2gwas -v [filename] -pf [filename] -p 1 -lmm -sv 7
```

The line will now be drawn at *-log10(1e-7)*.  
To disable the line and not label any SNPs, change the value to 0.

To remove the SNP lables completely, utilize the `-nl/--nolabel` option.  

```
vcf2gwas -v [filename] -pf [filename] -p 1 -lmm -nl
```

**Note**: This can be beneficial to reduce the overall runtime when the analysis results in many significant SNPs.  

To deactivate the manhattan and QQ-plots completely, employ the `-np/--noplot` option.

```
vcf2gwas -v [filename] -pf [filename] -p 1 -lmm -np
```

**Note**: This can be beneficial when using custom vcf files missing e.g. variant positions, or to reduce overall runtime.

### Change font size in all plots

Since the resulting plots may be used in various contexts, the font size of the plots produced by *vcf2gwas* can be changed by using the `-fs/--fontsize` option.

```
vcf2gwas -v [filename] -pf [filename] -p 1 -lmm -fs 20
```

The font sizes of the plots will now be changed from the default value of 26 pt to 20 pt.

### Keep temporary files

During the analysis, various temporary files like subsetted and filtered VCF and `.csv` files are produced. By default they are removed once they are no longer needed but if one wants to retain these files, employ the `-r/--retain` option:

```
vcf2gwas -v [filename] -pf [filename] -p 1 -lmm -r
```

## Output

The following part shows some of the output plots and summaries produced by the analysis of the example files using the linear mixed model and standard options.

### Quality Control

*vcf2gwas* will produce quality control plots (saved in the `QC` directory) for the phenotype data as well as the genotype data:  

* For every analyzed phenotype, its distribution will be plotted.
* For every chromosome of the genotype data, the raw variant density and, if available, DP (depth of coverage), MQ (mapping quality) and QD (Qual normalized by Depth) will be plotted.

To reduce the runtime, quality control can be deactivated with the option `-nq/--noqc`.  

### Plots

Manhattan-plot labeling significant SNPs with a standard significant value threshold of *-log10(1e-6)*:
<img src="https://github.com/frankvogt/vcf2gwas/blob/main/files/lmm_manh.png" alt="Manhattan-plot" width="75%"/>

QQ-plot comparing the expected and observed probability distributions:
<img src="https://github.com/frankvogt/vcf2gwas/blob/main/files/lmm_qq.png" alt="QQ-plot" width="75%"/>

### Summaries

Amongst other things, *vcf2gwas* will sort the SNPs of every analyzed phenotype, save the specified amount of top SNPs for each phenotype, summarize these SNPs of all phenotypes to check if certain SNPs occur more than once and optionally compare these SNPs to the genes supplied by the gene file. Below is the [output](https://github.com/frankvogt/vcf2gwas/blob/main/files/compared_summarized_top_SNPs_complete_example.csv) shown of such a gene comparison when supplying a [gene file](https://github.com/frankvogt/vcf2gwas/blob/main/files/NLR.csv) containing information about NLR genes in *A. thaliana*:

||||Upstream gene|Upstream gene|Upstream gene|Upstream gene||Downstream gene|Downstream gene|Downstream gene|Downstream gene|
|---|---|---|---|---|---|---|---|---|---|---|---|
|SNP ID|Chr|Phenotypes|ID|Comment|Name|Distance|SNP pos|Distance|Name|Comment|ID|
|3:2237364|3|avrRpm|AT3G07040.1|NB-ARC domain-containing disease resistance protein|RPM1|8340|2237364|||||
|3:2237394|3|avrRpm|AT3G07040.1|NB-ARC domain-containing disease resistance protein|RPM1|8370|2237394|||||
|3:2237446|3|avrRpm|AT3G07040.1|NB-ARC domain-containing disease resistance protein|RPM1|8422|2237446|||||
|3:2237452|3|avrRpm|AT3G07040.1|NB-ARC domain-containing disease resistance protein|RPM1|8428|2237452|||||
|3:2289171|3|avrRpm|AT3G07040.1|NB-ARC domain-containing disease resistance protein|RPM1|60147|2289171|||||
|2:975138|2|avrRpm|AT2G03030.1|Toll-Interleukin-Resistance (TIR) domain family protein||84618|975138|28330||Toll-Interleukin-Resistance (TIR) domain family protein|AT2G03300.1|
|2:975234|2|avrRpm|AT2G03030.1|Toll-Interleukin-Resistance (TIR) domain family protein||84714|975234|28234||Toll-Interleukin-Resistance (TIR) domain family protein|AT2G03300.1|
|2:975320|2|avrRpm|AT2G03030.1|Toll-Interleukin-Resistance (TIR) domain family protein||84800|975320|28148||Toll-Interleukin-Resistance (TIR) domain family protein|AT2G03300.1|
|2:975405|2|avrRpm|AT2G03030.1|Toll-Interleukin-Resistance (TIR) domain family protein||84885|975405|28063||Toll-Interleukin-Resistance (TIR) domain family protein|AT2G03300.1|
|2:975411|2|avrRpm|AT2G03030.1|Toll-Interleukin-Resistance (TIR) domain family protein||84891|975411|28057||Toll-Interleukin-Resistance (TIR) domain family protein|AT2G03300.1|
|2:975490|2|avrRpm|AT2G03030.1|Toll-Interleukin-Resistance (TIR) domain family protein||84970|975490|27978||Toll-Interleukin-Resistance (TIR) domain family protein|AT2G03300.1|
|2:975590|2|avrRpm|AT2G03030.1|Toll-Interleukin-Resistance (TIR) domain family protein||85070|975590|27878||Toll-Interleukin-Resistance (TIR) domain family protein|AT2G03300.1|
|1:11273854|1|avrRpm|||||11273854|14598|RAC1|Disease resistance protein (TIR-NBS-LRR class) family|AT1G31540.2|
|1:11273813|1|avrRpm|||||11273813|14639|RAC1|Disease resistance protein (TIR-NBS-LRR class) family|AT1G31540.2|
|3:2165688|3|avrRpm|||||2165688|60264|RPM1|NB-ARC domain-containing disease resistance protein|AT3G07040.1|

