# FINAL MUTATION CATALOGUE 2023 RESULT FILES - MAY 2024 UPDATES
Output files, including the VCF and the Excel files have been updapted as of May 2024.
We are grateful to @HillJamie who reported the issue with multiple consecutive nucleotide variants and synonymous model features.
When investigating the issue, we found another problem on some variants impacting terminal stop codon. Both issues have been fixed.

The main results of the catalogue Catalogue_master_file remain completely unchanged.

The issue was relatively minor and will not affect your pipeline if it does not consider multiple consecutive nucleotide variants nor synonymous changes.
For more details about the issue please reach out to tbsequencing@who.int

Previous files remain available by browsing the history of git commits, for reference.

# FINAL MUTATION CATALOGUE 2023 RESULT FILES - FEB 2024 UPDATES
Output files, including the VCF and the Excel files have been updapted as of February 2024.
We are grateful to the users who reported a small issue with multiple consecutive nucleotide variants and synonymous model features.

The main results of the catalogue Catalogue_master_file remain completely unchanged.

The issue was relatively minor and will not affect your pipeline if it does not consider multiple consecutive nucleotide variants nor synonymous changes.
For more details about the issue, please refer to the PDF presentation available in the Final Result Files folder and reach out to tbsequencing@who.int

Previous files remain available by browsing the history of git commits, for reference.

# FINAL MUTATION CATALOGUE 2023 RESULT FILES

Please find in the folder *Final Result Files* the main output of the mutation catalogue initiative, 2023 version.
This includes:
1. An Excel file with the complete data generated, including variant grades and all statistical metrics per variant
2. A VCF file matching genomic coordinates to variant names only
3. A PDF documenting their use

# RAW DATA FILES
All raw data used as input for the the "SOLO" algorithm can be found in the folder *Input data files for Solo algorithms*

### full_genotypes
Contains all variants (i.e. features of the models) for each sample.  
The data is arranged in two sub-levels of folders, one for drugs and one for gene tiers. The csv file contain 6 columns.

```
sample_id,resolved_symbol,variant_category,predicted_effect,neutral,"max(af)",position
```

- resolved_symbol is the gene name to which the variant belongs to
- variant_category is the final variant name. It is the result of the categorization algorithm which can be found in the *Genotype ETL - PySpark* folder.
- predicted_effect can have one of the following values:
1. feature_ablation (complete deletion)
2. frameshift
3. inframe_deletion
4. inframe_insertion
5. initiator_codon_variant (start codon changing to another start codon)
6. missense_variant
7. non_coding_transcript_exon_variant (non coding nucleotide changes, i.e. for *rrs/rrl*)
8. start_lost
9. stop_gained
10. stop_lost
11. stop_retained_variant (stop codon changing to another stop codon)
12. synonymous_variant
13. upstream_gene_variant
- max(af) is the percentage of reads showing the variant

position and neutral columns are not read by the algorithm and can be discarded

### phenotype
```
sample_id,phenotypic_category,phenotype,box
```

- phenotypic_category is the overall category the test was assigned to (`WHO` or `ALL`)
- phenotype is the result (`R` or `S`)
- box is the subcategory of the test. Refer to the mutation catalogue report for more details.

# CODE FILES

All other folders each include code used to generate the mutation catalogue. This includes:

## Bioinformatic pipeline - Step Functions
The [Step Function](https://aws.amazon.com/step-functions/) definition of our bioinformatic pipeline, in JSON format. Refer to the Step Function documentation for the specifications of the format. All executed commands are found in the *Parameters.Parameters.COMMAND* entry of each Step of the Worfklow.

## Variant Annotation - Python

We first created a custom reference database with `snpEff build` and using as input the GFF file from the NCBI Genome entry ID [GCA_000195955.2, Assembly version ASM19595v2](https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/195/955/GCF_000195955.2_ASM19595v2/GCF_000195955.2_ASM19595v2_genomic.gff.gz). We used the standard bacterial code of the NCBI genetic code tables. ([Table 11](https://www.ncbi.nlm.nih.gov/Taxonomy/Utils/wprintgc.cgi#SG11)).  

Then we used the following command to perform the annotation: 

```
snpEff ann -config snpEff.config -upDownStreamLen 2000 -spliceSiteSize 0 -spliceRegionExonSize 0 -spliceRegionIntronMax 0 -spliceRegionIntronMin 0 NC_000962.3 tmp.vcf | grep -v "^#" | cut -f3,8 > annotated_file.txt

```


Because of consistent errors of SnpEff, we curated, fixed and normalized its output using the scripts in the *Variant Annotation - Python* folder
The input file read by the script consists of two columns, with the first column being our internal variant id (integer) and the second column the `ANN` INFO tag of the VCF generated by `snpEff ann`.
The script can be modified to expect that column as output only if you wish to replicate our full annotation process.

## Solo algorithm - R implementation
Contains all scripts necessary to run the SOLO algorithm in R.

For issues related to this code, email Leonid Chindelevitch <lchindel@ic.ac.uk>

***
Before starting work please enusre you have installed all the required packages:

- *conflicted* 1.2.0 (for managing conflicting function names between packages) 
- *haven*      2.5.3 (for converting DTA files into CSV files; may be omitted)
- *igraph*     1.5.1 (for the graph-based implementation of the SOLO algorithm)
- *magrittr*   2.0.3 (for syntactic sugar in the code)
- *tidyverse*  2.0.0 (for data manipulation)

***
This folder contains various scripts required to execute the analysis that led
to version 2 of the catalogue, once the genomic variants have been extracted.

It implements a fully ported version of the SOLO algorithm that is at the core 
of the analysis, and provides an open-source alternative to the STATA codebase.
***
To unit-test the SOLO algorithm, run

```
setwd("Solo algorithm - R implementation")
source("Driver.R")
source("Constants.R")
source("SOLORefactoring.R")
stopifnot(testSOLO())
```

If no error message is produced, the unit test has been run successfully.
***
In additional to the phenotypic & genetic data extracted from our Postgres database,
a few complimentory files located in the folder "Input data files for Solo algorithms/additional-data"
are necessary to run a full replication of the algorithm 

***
A full replication of the analysis is run as follows; arguments explained below:

```
source("Solo algorithm - R implementation/Driver.R")

EXTRACTION_ID = "2023-04-25T06_00_10.443990_jr_b741dc136e079fa8583604a4915c0dc751724ae9880f06e7c2eacc939e086536"

mainDriver(
    EXTRACTION_ID = EXTRACTION_ID,
    OUTPUT_DIRECTORY = paste0("Results/", EXTRACTION_ID),
    SCRIPT_LOCATION_DIR = "Solo algorithm - R implementation",
    DATA_DIRECTORY = paste0("Input data files for Solo algorithms/", EXTRACTION_ID),
    NON_DATABASE_DIRECTORY = "Input data files for Solo algorithms/additional-data"
)
```


EXTRACTION_ID is the unique identifier for the preprocessed database containing 
genotypic and phenotypic information that serves as input for the analysis. The
default value corresponds to the database extraction provided in the repository.  

OUTPUT_DIRECTORY is the directory to be used for writing all the output files; 
by default, it is set to "Results/" followed by EXTRACTION_ID to make it unique.  

SCRIPT_LOCATION_DIR is the directory where all the R scripts including this one 
are located; the default value above works when run using the GitHub repository.  

DATA_DIRECTORY is the directory containing the database extraction;
the default value above works when run using the GitHub repository.  

NON_DATABASE_DIRECTORY is the directory containing the additional CSV files for
the analysis; the default value above works when run using the GitHub repository.  


## Solo algorithm - Stata implementation
Contains all scripts necessary to run the SOLO algorithm in Stata.

## Database Schema - Postgres
The definition of our  database which sustains all our analysis. Partly built using the [biosql](https://github.com/biosql/biosql) specifications.