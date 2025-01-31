---
title: "Genomic annotations"
author: "Mary Piper"
date: "Monday, March 4th, 2019"
---

Approximate time: 30 minutes

Learning Objectives:
-------------------

* Utilize genomic annotation databases to perform functional analysis on gene lists
* Understand the types of information stored in different databases 
* Explore the pros and cons of several popular R packages for retrieval of genomic annotations

# Genomic annotations

The analysis of next-generation sequencing results requires associating genes, transcripts, proteins, etc. with functional or regulatory information. To perform functional analysis on gene lists, we often need to obtain gene identifiers that are compatible with the tools we wish to use and this is not always trivial. Here, we discuss **ways in which you can obtain gene annotation information and some of advantages and disadvantages of each method**.

## Genome builds

When a new genome build is acquired, the names and/or coordinate location of genomic features (gene, transcript, exon, etc.) may change. Therefore, the annotations regarding genome features (gene, transcript, exon, etc.) is genome-build specific  and we need to make sure that our annotations are obtained from the appropriate resource. You should know which **build of the genome** was used to generate your gene list and make sure you use the **same build for the annotations** during functional analysis. 

For example, if we used the GRCh38 build of the human genome to quantify gene expression used for differential expression analysis, then we should use the **same GRCh38 build** of the genome to convert between gene IDs and to identify annotations for each of the genes. 

## Databases

We retrieve information on the processes, pathways, etc. for which a gene is involved, from the necessary database where the information is stored. The database you choose will be dependent on what type of information you are trying to obtain. Examples of databases that are often queried, include:

**General databases** 

Offer comprehensive information on genome features, feature coordinates, homology, variant information, phenotypes, protein domain/family information, associated biological processes/pathways, associated microRNAs, etc.:

- **Ensembl** (use Ensembl gene IDs)
- **NCBI** (use Entrez gene IDs)
- **UCSC** 
- **EMBL-EBI**

**Annotation-specific databases**

Provide annotations related to a specific topic:

- **Gene Ontology (GO):** database of gene ontology biological processes, cellular components and molecular functions - based on Ensembl or Entrez gene IDs or official gene symbols
- **KEGG:** database of biological pathways - based on Entrez gene IDs
- **MSigDB:** database of gene sets
- **Reactome:** database of biological pathways
- **Human Phenotype Ontology:** database of genes associated with human disease
- **CORUM:** database of protein complexes for human, mouse, rat
- **...**

This is by no means an exhaustive list, there are many other databases available that are not listed here.


## Tools

When performing functional analysis, the tools will take the list of genes you provide and retrieve information for each gene using one or more of these databases. Within R, there are many popular packages used for gene/transcript-level annotation:

**Interface tools:** for accessing/querying annotations from multiple different annotation databases

- **AnnotationDbi:** queries the *OrgDb*, *TxDb*, *Go.db*, *EnsDb*, and *BioMart* annotations.  
- **AnnotationHub:** queries large collection of whole genome resources, including ENSEMBL, UCSC, ENCODE, Broad Institute, KEGG, NIH Pathway Interaction Database, etc.

> **NOTE:** These are both packages that can be used to create the `tx2gene` files we had you download at the beginning of this workshop.

**Annotation tools:** for accessing/querying annotations from a specific database

- **org.*Gs*.eg.db:** these *OrgDb* annotation tools query gene feature information for the organism of interest, including gene IDs and associated GO and KEGG IDs - but unable to get previous gene build information easily
- **EnsDb.*Gspecies*.v86:** Ensembl database for transcript and gene-level information directly fetched from Ensembl API (similar to TxDb, but with filtering ability and versioned by Ensembl release) or can create using the *ensembldb* package
- **TxDb.*Gspecies*.UCSC.hg19.knownGene:** UCSC database for transcript and gene-level information or can create own *TxDb* from an SQLite database file using the *GenomicFeatures* package.
- **annotables:** easy-to-use package making gene-level feature information immediately available for the current and most recent genome builds for human/mouse
- **biomaRt:** wealth of information available by querying Ensembl's database using their [BioMart online tool](http://www.ensembl.org/biomart/martview/70dbbbe3f1c5389418b5ea1e02d89af3) - all previous genome builds and gene feature, structure, homology, variant, and sequence information available and connects to external databases; **however, currently being deprecated**


| Tool | Pros | Cons | Materials |
|:---:|:---|:---|:---:|
|**[org.Xx.eg.db](https://bioconductor.org/packages/release/bioc/vignettes/AnnotationDbi/inst/doc/IntroToAnnotationPackages.pdf)** | gene ID conversion, biotype and coordinate information | only latest genome build available | see below |
|**[EnsDb.Xx.vxx](http://bioconductor.org/packages/devel/bioc/vignettes/ensembldb/inst/doc/ensembldb.html)**| most up-to-date annotations, easy functions to extract features, direct filtering | more difficult to use than some packages | see below |
|**[TxDb.Xx.UCSC.hgxx.knownGene](https://bioconductor.org/packages/release/bioc/vignettes/GenomicFeatures/inst/doc/GenomicFeatures.pdf)** | feature information, easy functions to extract features | only available current and recent genome builds - can create your own, less up-to-date with annotations than Ensembl |
|**[annotables](https://github.com/stephenturner/annotables)** | super quick and easy gene ID conversion, biotype and coordinate information | only human & mouse, not updated regularly | no material available |
|**[biomaRt](https://bioconductor.org/packages/release/bioc/vignettes/biomaRt/inst/doc/biomaRt.html)** | all Ensembl database information available, all organisms on Ensembl, website & R interface | being deprecated | [link to material](https://github.com/hbctraining/Training-modules/blob/master/Ensembl-Biomart_R-based/lessons/biological_databases_Ensembl.md#ensembl-biomart) |

## AnnotationDbi

AnnotationDbi is an R package that provides an interface for connecting and querying various annotation databases using SQLite data storage. The AnnotationDbi packages can query the *OrgDb*, *TxDb*, *EnsDb*, *Go.db*, and *BioMart* annotations. There is helpful [documentation](https://bioconductor.org/packages/release/bioc/vignettes/AnnotationDbi/inst/doc/IntroToAnnotationPackages.pdf) available to reference when extracting data from any of these databases.

### org.Hs.eg.db

There are a plethora of organism-specific *orgDb* packages, such as `org.Hs.eg.db` for human and `org.Mm.eg.db` for mouse, and a list of organism databases can be found [here](https://www.bioconductor.org/packages/release/BiocViews.html#___OrgDb). These databases are best for converting gene IDs or obtaining GO information for current genome builds, but not for older genome builds. These packages provide the current builds corresponding to the release date of the package, and update every 6 months. If a package is not available for your organism of interest, you can create your own using *AnnotationHub*.

```r
# Load libraries
library(org.Hs.eg.db)
library(AnnotationDbi)

# Check object metadata
org.Hs.eg.db
```

We can see the metadata for the database by just typing the name of the database, including the species, last updates for the different source information, and the source urls. Note the KEGG data from this database was last updated in 2011, so may not be the best site for that information.

```r
OrgDb object:
| DBSCHEMAVERSION: 2.1
| Db type: OrgDb
| Supporting package: AnnotationDbi
| DBSCHEMA: HUMAN_DB
| ORGANISM: Homo sapiens
| SPECIES: Human
| EGSOURCEDATE: 2018-Oct11
| EGSOURCENAME: Entrez Gene
| EGSOURCEURL: ftp://ftp.ncbi.nlm.nih.gov/gene/DATA
| CENTRALID: EG
| TAXID: 9606
| GOSOURCENAME: Gene Ontology
| GOSOURCEURL: ftp://ftp.geneontology.org/pub/go/godatabase/archive/latest-lite/
| GOSOURCEDATE: 2018-Oct10
| GOEGSOURCEDATE: 2018-Oct11
| GOEGSOURCENAME: Entrez Gene
| GOEGSOURCEURL: ftp://ftp.ncbi.nlm.nih.gov/gene/DATA
| KEGGSOURCENAME: KEGG GENOME
| KEGGSOURCEURL: ftp://ftp.genome.jp/pub/kegg/genomes
| KEGGSOURCEDATE: 2011-Mar15
| GPSOURCENAME: UCSC Genome Bioinformatics (Homo sapiens)
| GPSOURCEURL: 
| GPSOURCEDATE: 2018-Oct2
| ENSOURCEDATE: 2018-Oct05
| ENSOURCENAME: Ensembl
| ENSOURCEURL: ftp://ftp.ensembl.org/pub/current_fasta
| UPSOURCENAME: Uniprot
| UPSOURCEURL: http://www.UniProt.org/
| UPSOURCEDATE: Thu Oct 18 05:22:10 2018
```

We can easily extract information from this database using *AnnotationDbi* with the methods: `columns`, `keys`, `keytypes`, and `select`. For example, we will use our `org.Hs.eg.db` database to acquire information, but know that the same methods work for the *TxDb*, *Go.db*, *EnsDb*, and *BioMart* annotations.

```r
# Return the Ensembl IDs for a set of genes
annotations_orgDb <- AnnotationDbi::select(org.Hs.eg.db, # database
                                     keys = res_tableOE_tb$gene,  # data to use for retrieval
                                     columns = c("SYMBOL", "ENTREZID","GENENAME"), # information to retreive for given data
                                     keytype = "ENSEMBL") # type of data given in 'keys' argument
```

This easily returned to us the information that we desired, but note the *warning* returned: *'select()' returned 1:many mapping between keys and columns*. This is always going to happen with converting between different gene IDs. Unless we would like to keep multiple mappings for a single gene, then we probably want to de-duplicate our data before using it.

```r
# Determine the indices for the non-duplicated genes
non_duplicates_idx <- which(duplicated(annotations_orgDb$SYMBOL) == FALSE)

# Return only the non-duplicated genes using indices
annotations_orgDb <- annotations_orgDb[non_duplicates_idx, ]
```

Note that if your analysis was conducted using an older genome (i.e hg19) some genes maybe found to be not annotated (NA), since orgDB is always the most recent release. It is likely that some of the genes have changed names in between versions (due to updates and patches), so may not be present in this version of the database. Our dataset was created based on the GRCh38 build of the human genome, using a recent release of Ensembl as our reference and so we should not see much of a discrepancy. 


### EnsDb.Hsapiens.v86

To generate the Ensembl annotations, the *EnsDb* database can also be easily queried using AnnotationDbi. You will need to decide the release of Ensembl you would like to query. All Ensembl releases are listed [here](http://useast.ensembl.org/info/website/archives/index.html). We know that our data is for GRCh38, and the most current release for GRCh38 in Bioconductor is release 86, so we can install this release of the *EnsDb* database. **NOTE: this is not the most current release, yet it is the latest release available through AnnotationDbi.**

Since we are using *AnnotationDbi* to query the database, we can use the same functions that we used previously:

```r
# Load the library
library(EnsDb.Hsapiens.v86)

# Check object metadata
EnsDb.Hsapiens.v86

# Explore the fields that can be used as keys
keytypes(EnsDb.Hsapiens.v86)
```

Now we can return all gene IDs for our gene list:

```r
# Return the Ensembl IDs for a set of genes
annotations_edb <- AnnotationDbi::select(EnsDb.Hsapiens.v86,
                                           keys = res_tableOE_tb$gene,
                                           columns = c("SYMBOL", "ENTREZID","GENEBIOTYPE"),
                                           keytype = "GENEID")
```

Then we can again deduplicate:

```r
# Determine the indices for the non-duplicated genes
non_duplicates_idx <- which(duplicated(annotations_edb$SYMBOL) == FALSE)

# Return only the non-duplicated genes using indices
annotations_edb <- annotations_edb[non_duplicates_idx, ]
```

## AnnotationHub

AnnotationHub is a wonderful resource for accessing genomic data or querying large collection of whole genome resources, including ENSEMBL, UCSC, ENCODE, Broad Institute, KEGG, NIH Pathway Interaction Database, etc. All of this information is stored and easily accessible by directly connecting to the database.

To get started with AnnotationHub, we first load the library and connect to the database:

```r
# Load libraries
library(AnnotationHub)
library(ensembldb)

# Connect to AnnotationHub
ah <- AnnotationHub()
```

To see the types of information stored inside, we can just type the name of the object:

```r
# Explore the AnnotationHub object
ah
```

Using the output, you can get an idea of the information that you can query within the AnnotationHub object:

```
AnnotationHub with 47474 records
# snapshotDate(): 2018-10-24 
# $dataprovider: BroadInstitute, Ensembl, UCSC, ftp://ftp.ncbi.nlm.nih.gov/gene/DATA/, Haemcode, Inparanoid8, FungiDB, ...
# $species: Homo sapiens, Mus musculus, Drosophila melanogaster, Bos taurus, Pan troglodytes, Rattus norvegicus, Danio ...
# $rdataclass: GRanges, BigWigFile, FaFile, TwoBitFile, OrgDb, Rle, ChainFile, EnsDb, Inparanoid8Db, TxDb
# additional mcols(): taxonomyid, genome, description, coordinate_1_based, maintainer, rdatadateadded,
#   preparerclass, tags, rdatapath, sourceurl, sourcetype 
# retrieve records with, e.g., 'object[["AH2"]]' 

            title                                               
  AH2     | Ailuropoda_melanoleuca.ailMel1.69.dna.toplevel.fa   
  AH3     | Ailuropoda_melanoleuca.ailMel1.69.dna_rm.toplevel.fa
  AH4     | Ailuropoda_melanoleuca.ailMel1.69.dna_sm.toplevel.fa
  AH5     | Ailuropoda_melanoleuca.ailMel1.69.ncrna.fa          
  AH6     | Ailuropoda_melanoleuca.ailMel1.69.pep.all.fa        
  ...       ...                                                 
  AH67895 | org.Vibrio_vulnificus.eg.sqlite                     
  AH67896 | org.Achromobacter_group_B.eg.sqlite                 
  AH67897 | org.Achromobacter_group_E.eg.sqlite                 
  AH67898 | org.Pannonibacter_phragmitetus.eg.sqlite            
  AH67899 | org.Salinispora_arenicola_CNS-205.eg.sqlite
  ```
  
Notice the note on retrieving records with `object[[AH2]]` - this will be how we can extract a single record from the AnnotationHub object.
  
If you would like to see more information about any of the classes of data you can extract that information as well. For example, if you wanted to determine all species information available, you could subset the AnnotationHub object:
  
```r
# Explore all species information available
unique(ah$species)
```
  
Now that we know the types of information available from AnnotationHub we can query it for the information we want using the `query()` function. Let's say we would like to return the Ensembl `EnsDb` information for human. To return the records available, we need to use the terms as they are output from the `ah` object to extract the desired data.
  
```r
# Query AnnotationHub
human_ens <- query(ah, c("Homo sapiens", "EnsDb"))
```

The output for the `EnsDb` objects is much more recent than what we encountered with AnnotationDbi (most current release is Ensembl 94), however for Homo sapiens the releases only go back as far as Ensembl 87. This is fine if you are using GRCh38, however if you were using an older genome build like hg19, you would need to load the `EnsDb` package if available for that release or you might need to build your own with `ensembldb`.

In our case, we are looking for the latest Ensembl release so that the annotations are the most up-to-date. To extract this information from AnnotationHub, we can use the AnnotationHub ID to subset the object:

```r
# Extract annotations of interest
human_ens <- human_ens[["AH64923"]]
```

Now we can use `ensembldb` functions to extract the information at the gene, transcript, or exon levels. We are interested in the gene-level annotations, so we can extract that information as follows:

```r
# Extract gene-level information
genes(human_ens, return.type = "data.frame") %>% View()
```
But note that it is just as easy to get the transcript- or exon-level information:

```r
# Extract transcript-level information
transcripts(human_ens, return.type = "data.frame") %>% View()

# Extract exon-level information
exons(human_ens, return.type = "data.frame") %>% View()
```

### Using AnnotationHub to create our tx2gene file

To create our `tx2gene` file, we would need to use a combination of the methods above and merge two dataframes together. For example:

```r
# Create a transcript dataframe
 txdb <- transcripts(human_ens, return.type = "data.frame") %>%
   dplyr::select(tx_id, gene_id)
 txdb <- txdb[grep("ENST", txdb$tx_id),]
 
 # Create a gene-level dataframe
 genedb <- genes(human_ens, return.type = "data.frame")  %>%
   dplyr::select(gene_id, symbol)
 
 # Merge the two dataframes together
 annotations <- inner_join(txdb, genedb)

```

In this lesson our focus has been using annotation packages to extract information mainly just for gene ID conversion for the different tools that we use downstream. Many of the annotation packages we have presented have much more information than what we need for functional analysis and we have only just scratched the surface here. It's good to know the capabilities of the tools we use, so we encourage you to spend some time exploring these packages to become more familiar with them.


> **NOTE:** The *annotables* package is a super easy annotation package to use. It is not updated frequently, so it's not great for getting the most up-to-date information for the current builds and does not have information for other organisms than human and mouse, but is a quick way to get annotatio information. 
>
>```r
># Load library
>library(annotables)
>
># Access previous build of annotations
>grch38
>```



---
*This lesson has been developed by members of the teaching team at the [Harvard Chan Bioinformatics Core (HBC)](http://bioinformatics.sph.harvard.edu/). These are open access materials distributed under the terms of the [Creative Commons Attribution license](https://creativecommons.org/licenses/by/4.0/) (CC BY 4.0), which permits unrestricted use, distribution, and reproduction in any medium, provided the original author and source are credited.*
