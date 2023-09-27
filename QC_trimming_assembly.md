# Installing and running softwares on ARC using conda and pip
## Filtlong
```
conda create --name filtlong -c bioconda filtlong #made new env called filtlong
conda activate filtlong
```
To run filtlong
```
filtlong --min_mean_q 10 passed_reads.fastq.gz| gzip > test.fastq.gz
```
## Flye
```
conda install -c bioconda flye
flye -
```
## Medaka
```
pip install medaka
```
