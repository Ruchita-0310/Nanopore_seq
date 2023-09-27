# Installing and running softwares on ARC using conda and pip
## Filtlong
```
conda create --name filtlong -c bioconda filtlong
conda activate filtlong


```
To run filtlong
```
../Filtlong/bin/filtlong passed_reads.fastq.gz --min_mean_q 7 | gzip > filtlong_output.fastq 
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
