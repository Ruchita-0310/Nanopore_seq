# Samtools
It will be important for you to convert .sam files to .bam files and indexing .bam files. 
```
conda create -n samtools-env
conda activate samtools-env
conda install -c bioconda samtools
samtools view -Sb sam_file_name.sam -o bam_file_name.bam
samtools sort -o sort.bam bam_file_name.bam --write-index
```
