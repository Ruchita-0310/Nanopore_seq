# Installing and running softwares on ARC using conda and pip
## Filtlong
```
conda create --name filtlong -c bioconda filtlong #made new env called filtlong
conda activate filtlong
```
To run filtlong
```
filtlong --min_mean_q 7 passed_reads.fastq.gz| gzip > test.fastq.gz #will produce test.fastq.gz file which will be used for nanoplot
```
## Nanoplot
```
module load python/3.10.4
pip3 install NanoPlot
nano nanoplot_test.sbatch #job script
#!/bin/bash
####### Reserve computing resources #############
#SBATCH --nodes=1
#SBATCH --ntasks=2
#SBATCH --cpus-per-task=4
#SBATCH --time=24:00:00
#SBATCH --mem=1G
#SBATCH --partition=cpu2019
####### Run your script #########################
NanoPlot -t 8 --fastq test.fastq.gz --maxlength 4000000 --plots dot --legacy hex 
```
## Flye
```
conda install -c bioconda flye #in filtlong env
flye -
```
## Medaka
```
pip install medaka #in filtlong env
```
