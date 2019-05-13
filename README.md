## SCANVIS

SCANVIS is a set of tools for **SC**oring, **AN**notating and **VIS**ualizing splice junctions using gencode annotation. It scores splice junctions by using a Relative Read Support (RRS) measure that relates the reads supporting a query junction to reads supporting nearby annotated splice junctions. It annotates each splice junction, indicating whether it is supported by the annotation or not and what genes overlap the junction. For unannotated junctions, details on the junction type and whether it is in-frame or not are also provided. SCANVIS also has a visualization component that allows users to quickly view one or more samples in sashimi style plots, showing splice junctions and, optionally, a read coverage profile and/or mutations in one figure (see example [here](PPA2.LUSC.exon_skip.pdf)). These sashimi style plots are novel in that unannotated splice junctions are highlighted in various colours to delineate different junction types, with line styles indicating whether unannotated junctions are in frame or not, and junction arc heights and thickness corresponding to read support and RRS scores respectively. For more details on the software and usage, please see our paper and the [SCANVIS Manual](SCANVIS-manual.pdf).

* Version: 0.99.15
* Author: Phaedra Agius, [New York Genome Center](https://www.nygenome.org)
* Email:  <pagius@nygenome.org> 

SCANVIS is freely available for academic and non-commercial research purposes only ([License](LICENSE))


## Installation

To install directly from github:

`install.packages("devtools")`  
`devtools::install_github("nygenome/SCANVIS")`

Alternatively you can download the gz tar and install on R>=3.5.0 by executing the following command:

`install.packages('SCANVIS_0.99.0.tar.gz')`

### Dependencies
Installation of SCANVIS requires the following R packages: IRanges,plotrix,RCurl,rtracklayer

## Basic Usage

SCANVIS has six main functions: **SCANVIS.annotation, SCANVIS.scan, SCANVIS.linkvar, SCANVIS.merge, SCANVIS.visual, SCANVIS.read_STAR**.
The **scan**, **linkvar** and **visual** SCANVIS functions require a gencode object generated by **SCANVIS.annotation** with the object supporting human gencode19, samples of which are included in the examples. For a full version of gencode 19 or for any other gencode version, we recommend users issue the following command using a suitable ftp url (Eg. ftp://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_29/) to a gencode directory of choice:

`gen=SCANVIS.annotation(<FTP.URL>)`

The next step is to score and annotate a set of splice junctions using the **SCANVIS.scan** function. A <SJ> 4 column matrix be prepared with columns labeled as "chr", "start", "end", "uniq.reads" to indicate the genomic coordinates and read support for the junctions. Note that such data is standard output from the [STAR](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4631051/) alignment software, and the **SCANVIS.read_STAR** can be used to read in STAR output as required for **SCANVIS.scan**. However users may derive SJ details from any alignment software of choice. To score and annotate SJs, execute the following command:

`scn=SCANVIS.scan(SJ,gen)`

If the user requires Relative Read Coverage scores for novel exons (see manuscript for more details), then we recommend running **SCANVIS.scan** by supplying urls to the bamfile and executable samtools likeso:

`scn=SCANVIS.scan(<SJ>,gen,<BAM>,<SAMTOOLS>)`

Users may now (optionally) map variants to the **SCANVIS.scan** output using the **SCANVIS.linkvar** function. A variant file <VARIANTS> must be prepared in bed format as a four column matrix with columns 1-3 being labeled "chr", "start", "end". The fourth column name should be a description of the variant type (eg. "ssMUT" would be a good title for splice site mutations) with entries describing the variant however users wish (eg. GT>C or rs123456). Variants can then be mapped to SJs by issuing the following command:

`scnv=SCANVIS.linkvar(scn,<VARIANTS>,gen,p)`

where `p` is a user defined parameter that relaxes/expands variant intervals by *p* base pairs, giving them a better chance
to overlap SJ intervals (default: *p=0*).

While SCANVIS processes one sample at a time and generates output accordingly, users may wish to assemble PSI scores or SJ read supports in a single matrix in order to compare samples. **SCANVIS.merge** is the function for this. Users supply a number of SCANVIS outputs (either urls or matrices in list format) and the function generates a matrix containing PSI scores for all samples across the union of SJs is assembled, a similar matrix with supporting SJ reads, and a mutation matrix (binary) if the supplied SJ files are variant mapped. A representative sample is also computed and can be visualized - this is assembled using the mean (or median, user-defined by the `method` option) of supporting SJ reads and PSI scores across all samples.

`scn_mult=list('s1'=scn1,'s2'=scn2,...'sN'=scnN)`  
`scn_mult=c('~/pathtosample1','~/pathtosample2',...'~/pathtosampleN')`  
`scn_merged=SCANVIS.merge(scn_mult,method='mean',roi='chr1')`

Finally users may view sashimi-style plots for a SCANVIS output of a gene name <GENE> or region of interest <ROI> (3 bit vector chr,start,end) by executing one of the following:

`vis.out=SCANVIS.visual(<GENE>,gen,scn)`  
`vis.out=SCANVIS.visual(<ROI>,gen,scn)`

Users may also supply a set of samples in which case the function automatically calls upon **SCANVIS.merge** to generate a representative sample, and a sashimi visual is then generated. If the samples supplied are variant mapped SJs, then variants are also shown in the plot. As an example, the manual shows users how to plot SCANVIS figures for two LUSC samples from TCGA that harbored a uniquely (unique within our TCGA cohort) occuring exon skipping event, with one of the samples having a splice site variant right at the skipped exon. 
There are a few parameters to allow users to control figure details such as inserting a title, expand gene annotations/isoforms and even highlight SJs of interest (useful for select annotated SJs as these are generally harder to distinguish in the plot). More details on parameter choices can be found in the manual [here](SCANVIS-manual.pdf). 
