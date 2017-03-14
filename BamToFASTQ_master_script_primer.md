
# Batch Conversion of bam to fastq files

#### A primer to creating a master shell script which creates sample specific scripts for batch conversion of bam to paired end FASTQ files.

TO DO : 
1. pre-face
2. in depth explanation of code

#### Project Directory - The directory tree of where the raw data is stored (away from the home user directory as indicated by '...' )

```
...ProjectData/
├── DatasetOne/
    ├── sample1/
        └── sample1.bam
    ├── sample2/
        └── sample2.bam
    ├── ...
    └──  sample100/
        └── sample100.bam
└── DatasetTwo/
    ├── sample1/
        └── sample1.bam
    ├── sample2/
        └── sample2.bam
    ├── ...
    └──  sample100/
        └── sample100.bam
```

#### Home Directory - Before running BamToFASTQ_master.sh

How the home directory would be set up for analysis.

```
MyProject/
├── BamToFASTQ_master.sh
├── scripts/
├── analysis/
├── references/
└── data/
```
The master shell script :

```bash
#!/bin/bash
# author : delvin so
# not intended to be qsubbed so no parameters for SGE
# assuming .bam filenames are identical to the folder name


echo "Starting BamToFastQ Batch Conversion"
date

# setting the appropriate input, output and script directories depending on dataset I'm working with

DATASET=ONE
if [ $DATASET = 'ONE' ]; then
  echo "###################### Settings for DATASET ONE ######################"
  INDIR='/ProjectData/DatasetOne'
  OUTDIR='/MyProject/data/DatasetOne'
  SCRIPTDIR='/MyProject/scripts/DatasetOne/BAMtoFASTQ'
else
  echo "###################### Settings for DATASET TWO ######################"
  INDIR='/ProjectData/DatasetTwo'
  OUTDIR='/MyProject/data/DatasetTwo'
  SCRIPTDIR='/MyProject/scripts/DatasetTwo/BAMtoFASTQ'
fi

for file in $INDIR/*.*
  do
  # storing the filename from the .bam file and removing the suffix
  fileName=$(basename "$file" .bam)

  mkdir $OUTDIR'/'$fileName #creating a directory in the out directory with the .bam file name

  # tailoring and creating the script for each individual sample
  echo "
#$ -N $fileName'_BamToFastQ'
#$ -o :$HOME/Logs/$fileName'.out.log'
#$ -j y
#$ -m bea -M so.delvin@gmail.com

module load java/8
module load picard

java -jar $picard_dir/picard.jar SamToFastq \
I=$file \
FASTQ=$OUTDIR'/'$fileName'/'$fileName'_read1.fastq' \
SECOND_END_FASTQ=$OUTDIR'/'$fileName'/'$fileName'_read2.fastq'
" >> $SCRIPTDIR/$fileName.sh

  chmod +X  $SCRIPTDIR/$fileName.sh
  qsub  $SCRIPTDIR/$fileName.sh
  fi
  done
```
#### Home directory - after executing BAMtoFASTQ_master.sh

```
  MyProject/
  ├── BamToFASTQ_master.sh
  ├── scripts/
      ├── DatasetOne/
          ├── BAMtoFASTQ/
            ├── sample1.sh
            ├── sample2.sh
            ├── sample3.sh
            ├── ...
            └── sample100.sh
      └── DatasetTwo/
          ├── BAMtoFASTQ/
            ├── sample1.sh
            ├── sample2.sh
            ├── sample3.sh
            ├── ...
            └── sample100.sh
  ├── analysis/
  ├── references/
  └── data/
      ├── DatasetOne/
          ├── sample1/
              ├── sample1_read1.fastq
              └── sample1_read2.fastq
          ├── ...
          └── sample100/
              ├── sample100_read1.fastq
              └── sample100_read2.fastq
      └── DatasetTwo/
          ├── sample1/
              ├── sample1_read1.fastq
              └── sample1_read2.fastq
          ├── ...
          └── sample100/
              ├── sample100_read1.fastq
              └── sample100_read2.fastq
```
