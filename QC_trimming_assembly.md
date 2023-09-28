# Installing and running softwares on ARC using conda and pip
## Filtlong
```
conda create --name filtlong -c bioconda filtlong #made new env called filtlong
conda activate filtlong
```
To run filtlong
```
filtlong --min_mean_q 9 passed_reads.fastq.gz| gzip > test_2.fastq.gz #will produce test_2.fastq.gz file which will be used for nanoplot
```
Setting min_mean_q to 7 ensures that reads with an average quality score below 7 will be filtered out. This helps remove reads that are more
likely to contain sequencing errors or low-quality data, which can negatively impact downstream analyses such as genome assembly or variant
calling.
## Nanoplot
```
module load python/3.10.4
pip3 install NanoPlot
nano nanoplot_test.sbatch #creating a job script
#!/bin/bash
####### Reserve computing resources #############
#SBATCH --nodes=1
#SBATCH --ntasks=2
#SBATCH --cpus-per-task=4
#SBATCH --time=24:00:00
#SBATCH --mem=1G
#SBATCH --partition=cpu2019
####### Run your script #########################
NanoPlot -t 8 --fastq test_2.fastq.gz --maxlength 4000000 --plots dot --legacy hex
sbatch nanoplot_test.sbatch #command to run job script
```
## Flye
```
conda install -c bioconda flye #in filtlong env
#!/bin/bash
####### Reserve computing resources #############
#SBATCH --nodes=1
#SBATCH --ntasks=2
#SBATCH --cpus-per-task=4
#SBATCH --time=72:00:00
#SBATCH --mem=15G
#SBATCH --partition=cpu2019
####### Run your script #########################
flye --nano-raw test.fastq.gz --meta --genome-size 50m --out-dir assembly_flye -i 0 --threads 8
```
## Medaka
```
pip install medaka #in filtlong env
```
