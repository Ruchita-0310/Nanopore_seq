# Installing and running softwares on ARC using conda and pip
## Filtlong
```
git clone https://github.com/rrwick/Filtlong.git
cd Filtlong
make -j
bin/filtlong -h
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
