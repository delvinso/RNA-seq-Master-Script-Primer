# RNA-seq-Master-Script-Primer
## A Primer for Isoform Expression Analysis using RNA-Seq Reads with the Tuxedo Suite

This document is intended to serve as a quick guide to creating a master shell script for batch analysis for RNA-seq paired end reads and assumes you have
some background of bash commands, RNA-seq and working on a server cluster. **The script does not apply only to the Tuxedo Suite and can be customized for any large scale sample analysis.**

## The Problem

We want to perform isoform expression analysis of several hundred common cancer cell lines between two datasets (or any other two conditions of interests), the directory tree for said project is below.
Given an experiment where there are hundreds of cell lines in this experiment (or any other), no one wants to manually create a script or submit them one at a time.
This documents provides a general overview of how to create a master shell script, which automatically creates a catered, specific script that provides the necessary commands to process each sample
with bowtie2 and tophat2, providing output for isoform expression analysis.

### Directory Tree of Experiment
For this example, assume the data for the experiment is kept in a separate directory from your home. In this folder are two subdirectories, each containing further subdirectories with
samples and their corresponding paired end reads in .fastq format.

```
...ProjectData/
├── DatasetOne/
    ├── sample1/
        ├── sample1_read1.fastq
        └── sample1_read2.fastq
    ├── sample2/
        ├── sample2_read1.fastq
        └── sample2_read2.fastq
    ├── ...
    └──  sample100/
        ├── sample100_read1.fastq
        └── sample100_read2.fastq
└── DatasetTwo/
    ├── sample1/
        ├── sample1_read1.rnaseq.fastq.gz
        └── sample1_read2.rnaseq.fastq.gz
    ├── sample2/
        ├── sample2_read1.rnaseq.fastq.gz
        └── sample2_read2.rnaseq.fastq.gz
    ├── ...
    └──  sample100/
        ├── sample100_read1.rnaseq.fastq.gz
        └── sample100_read2.rnaseq.fastq.gz
```


### Home Directory - Before running master_tophat2_cufflinks.sh
```
How the home directory would be set up for analysis.
MyProject/
├── master_tophat2_cufflinks.sh
├── scripts/
├── analysis/
    ├──tophat2/
    └──cufflinks/
├── references/
      └── Homo_Sapiens/
          └──GRCh38/
              ├──genes.gtf
              ├──Bowtie2Index/
                 └── [bt2 index files]
              ├──transcriptome-index/
                 └── [transcriptome index files ]
              └──WholeGenomeFasta/
                 └── genome.fa
```

### The contents of the master shell script

```bash
#!/bin/bash
date


#Location of reference files for tophat2 and cufflinks

BOWTIE2INDEX=/MyProject/references/Homo_Sapiens/GRCh38/Bowtie2Index/genome
GENE_REFERENCE=/MyProject/references/Homo_Sapiens/GRCh38/genes.gtf
TRANSCRIPTOMEDIR=/MyProject/references/Homo_Sapiens/GRCh38/transcriptome-index/genes
REF=/MyProject/references/Homo_Sapiens/GRCh38/WholeGenomeFasta/genome.fa

#setting the appropriate input, output and script directories depending on dataset I'm working with
#don't have to manually change the variables and I know I'll only be working with these two datasets
DATASET=ONE
if [ $DATASET = 'ONE' ]; then
  echo "###################### Settings for DATASET ONE ######################"
  TH2_INDIR=.../ProjectData/DatasetOne/sample1
  TH2_OUTDIR=/MyProject/data/DatasetOne/analysis/tophat2
  CL_OUTDIR=/MyProject/data/DatasetOne/analysis/cufflinks
  SCRIPTDIR=/MyProject/scripts/DatasetOne
else
  echo "###################### Settings for DATASET TWO ######################"
  TH2_INDIR=.../ProjectData/DatasetTwo/sample2
  TH2_OUTDIR=/MyProject/data/DatasetTwo/analysis/tophat2
  CL_OUTDIR=/MyProject/data/DatasetTwo/analysis/cufflinks
  SCRIPTDIR=/MyProject/scripts/DataSetTwo
fi

for file in $TH2_INDIR/*;
  do

  # grab basename of current cell line to name output files
  fileName=$(basename $file)
  echo $fileName

  # creating output directory for each cell line so each cell line has its own folder under its corresponding analysis
  [ ! -d $TH2_OUTDIR/$fileName ] && mkdir $TH2_OUTDIR/$fileName
  [ ! -d $CL_OUTDIR/$fileName ] && mkdir $CL_OUTDIR/$fileName

  echo "
#!/bin/bash
#$ -N _$fileName'_tophat2_cufflinks'
#$ -o :$HOME/Logs/$fileName'.out.log'
#$ -j y
#$ -m bea -M so.delvin@gmail.com
#$ -pe smp 4 -l h_vmem=4G

module load bowtie2/2.2.4
module load tophat2/2.1.0
module load cufflinks/2.2.1

# tophat2 parameters

tophat2 -G $GENE_REFERENCE \
-p 4 \
--no-coverage-search \
--mate-inner-dist 150 \
--mate-std-dev 50 \
--transcriptome-index=$TRANSCRIPTOMEDIR \
-o $TH2_OUTDIR/$fileName \
$BOWTIE2INDEX \
$TH2_INDIR/$fileName/$fileName'_read1.'* $TH2_INDIR/$fileName/$fileName'_read2.'*

# renaming the tophat2 output bam file with corresponding cell line name for easier identificatiion down the road
mv $TH2_OUTDIR/$fileName/accepted_hits.bam $TH2_OUTDIR/$fileName/accepted_hits_$fileName.bam

# cufflinks parameters

cufflinks -G $GENE_REFERENCE \
-p 6 \
-o $CL_OUTDIR/$fileName \
-b $REF \
$TH2_OUTDIR/$fileName/accepted_hits_$fileName.bam
" > $SCRIPTDIR/_$fileName.sh

  chmod +x $SCRIPTDIR/_$fileName.sh
  qsub $SCRIPTDIR/_$fileName.sh
  done
```

### Home - After running the master shell script
```
MyProject/
├── master_tophat2_cufflinks.sh
├── scripts/
    ├──DatasetOne/
       ├── sample1.sh
       ├── sample2.sh
       ├── ...
       └──sample100.sh
    └──DatasetTwo/
       ├── sample1.sh
       ├── sample2.sh
       ├── ...
       └──sample100.sh
├── data/
├── Logs/
    ├── sample1.out.log
    ├── sample2.out.log
    ├── ...
    └──sample100.out.log
└──analysis/
    ├──tophat2
       ├── sample1/
           └── [ tophat2 output files ]
       ├── sample2/
           └── [ tophat2 output files ]
       ├── ...
       └──sample100/
           └── [ tophat2 output files ]
    └──cufflinks
       ├── sample1/
           └── [ cufflinks output files ]
       ├── sample2/
           └── [ cufflinks output files ]
       ├── ...
       └── sample100/
           └── [ cufflinks output files ]
├── references/
    └── Homo_Sapiens/
        └──GRCh38/
            ├──genes.gtf
            ├──Bowtie2Index/
            ├──transcriptome-index/
            └──WholeGenomeFasta/
```

### Breaking down the master shell script.... (WIP)
Could do something where I explain each component of the script and then have the script in full at the end??
1. Setting the appropriate variables and directory paths
2. Tophat2 - alignment
3. Cufflinks - expression quantification
4. ....

