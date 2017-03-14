
# INCOMPLETE : A primer to creating a master shell script to retrieve file directories and generate symbolic links.

Useful when there is a specific subset of files that you would like to look at without having to manually search through your project data files.
For example, your project directory contains every possible cancer cell line, but you want to specifically look at only pancreatic cancer cell lines and have the identifiers to such cell lines.

TO DO:
1. Creating example before/after directories
2. Touch up the comments as most of this is masked from my lab so explanations may be uninformative or vague.
3. Explain code in detail. 


### What does this script aim to do?
1. Takes in a text file with sample identifiers of interest as input and searches the respective folder (Dataset One or Two) for the RNA-seq reads.
2. Creates an intermediate text file with the directory path containing the samples of interest.
3. Using the intermediate text file, creates symbolic links to each of the samples of interest in the specified output directory.
#### NOTE : Slightly more complicated for dataset two due to having to map the appropriate sample identifier to folder and file names as they RNA-seq reads are not correspondingly named.

```bash
#!/bin/bash
#not intended to be qsubbed so no server arguments


DATASET=ONE
#Dataset Two settings are WIP - integrate mapping to sample identifiers for easier .fastq retrieval

if [ $DATASET= 'ONE' ]; then
  echo "###################### Settings for ONE ######################"
  SEARCHDIR=/ProjectData/DatasetTwo #Directory to search
  OUTDIR=/MyProject/data/DatasetOne  #Where the symbolic links will be stored, DON"T FORGET TO MAKE THIS DIRECTORY!!
  QUERYLIST=??????? #text file with sample identifiers of interest
  DIRECTORYLIST=~/itworks.txt #location and filename of textfile storing directory listing

  cd $SEARCHDIR
  find -L `pwd` -name \*.bam | grep -f $QUERYLIST >$DIRECTORYLIST

  while read file;
  do
    fileName=$(basename $file .bam)
    ln -s $file $OUTDIR/$fileName
    echo "Symbolic link to " $file "created"
  done <$DIRECTORYLIST

fi
elif [ $DATASET= 'TWO' ]; then
  #1.use map file, look for 
  line identifier in map file, find id number
  #2.grep corresponding id number folders
  # cat Run_Sample_meta_info.map | cut -f2,5 | cut -f1 -d ';' | sed -e 's/Cell_line=//g'
  echo "###################### Settings for TWO ######################"
  SEARCHDIR=/ProjectData/DatasetOne #Directory to search
  DATASETTWOMAPLIST=/MyProject/mapped_identifiers.txt
  QUERYLIST=/MyProject/querylist.txt #Text file with filenames of interest -
  ADJUSTEDLIST=/MyProject/adjustedlist.txt #directory of adjusted list
  OUTDIR=~/MyProject/data/DatasetTwo
  DIRECTORYLIST=~/directorylist.txt

  #need to check below
  cat $DATASETTWOMAPLIST | grep -f $QUERYLIST | cut -f1 > $ADJUSTEDLIST #searching mapped list with querylist of sample identifiers and storing in a new file
                                                                  #accounting for identifiers associated with sample

  cd $SEARCHDIR
  find -L `pwd` -name \*.fastq* | grep -f $ADJUSTEDLIST > $DIRECTORYLIST

  #making folders
  # while read folders;
  # do
  # mkdir $OUTDIR/$folders
  # done<$ADJUSTEDLIST
  #NEED CORRESPONDING FOLDERS FOR FASTQ FILES OR ELSE ALL FILES STORED IN ONE DIRECTORY.
  #Could work but need to adjust th2/cl master script.

  while read file;
  do
  fileName=$(basename $file .fastq)
  ln -s $file $OUTDIR/$fileName/
  echo "Symbolic link for $fileName created"
  done <$DIRECTORYLIST

fi
```
