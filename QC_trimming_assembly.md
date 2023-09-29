# Installing and running softwares on ARC using conda and pip
## 1. Filtlong
[Filtlong](https://github.com/rrwick/Filtlong) a tool for filtering long reads
```
conda create --name filtlong -c bioconda filtlong #made new env called filtlong
conda activate filtlong
filtlong --min_mean_q 80 passed_reads.fastq.gz| gzip > test_2.fastq.gz #will produce test_2.fastq.gz file which will be used for downstream analysis
```
[Mean q](https://github.com/rrwick/Filtlong#read-scoring) is set to 80 to remove the reads that were less than 80% correct which was already done by guppy. So you can play around with different mean_q values to get a better assembly. 
I used 80% and 95% and will compared it using flye. 
95% is too high to get circular genomes. So keep mean q to 80%. 
## 2. Nanoplot
[NanoPlot](https://github.com/wdecoster/NanoPlot) is a good tool to visualize the data
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
## 3. Flye
[Flye](https://github.com/fenderglass/Flye) is an assembler 
```
conda install -c bioconda flye #in filtlong env
#!/bin/bash
####### Reserve computing resources #############
#SBATCH --nodes=1
#SBATCH --ntasks=2
#SBATCH --cpus-per-task=4
#SBATCH --time=24:00:00
#SBATCH --mem=15G
#SBATCH --partition=bigmem
####### Run your script #########################
flye --nano-raw test_2.fastq.gz --meta --genome-size 15m --out-dir assembly_flye -i 0 --threads 8
```
## 4. Medaka
[Medaka](https://github.com/nanoporetech/medaka) a tool to create consensus sequences
```
pip install medaka #in filtlong env
```
