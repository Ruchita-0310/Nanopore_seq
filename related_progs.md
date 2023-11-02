# Samtools
It will be important for you to convert .sam files to .bam files and indexing .bam files. 
```
conda create -n samtools-env
conda activate samtools-env
conda install -c bioconda samtools
samtools view -Sb sam_file_name.sam -o bam_file_name.bam
samtools sort -o sort.bam bam_file_name.bam --write-index
```
# CheckM2
```
git clone --recursive https://github.com/chklovski/checkm2.git
cd checkm2
conda env create -n checkm2 -f checkm2.yml
conda activate checkm2
python setup.py install
conda activate checkm2

mamba create -n checkm2 -c bioconda -c conda-forge checkm2
mamba activate checkm2
mamba install python=3.8
checkm2 database --download
```
