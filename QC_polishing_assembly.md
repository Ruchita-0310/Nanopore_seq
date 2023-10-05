# Quality control
Before doing any quality control, concatenate all the passed reads into one file 
```
cat ./fastq_pass/*.fastq.gz > passed_reads.fastq.gz
```
```
squeue -u ruchita.solanki #command to see what is running
```
## 1. Porechop
[Porechop](https://github.com/rrwick/Porechop) eventhough it is no longer available, you could still use
it. 
Guppy basecaller does the adapter trimming. Still better to use porechop. 
```
conda create -n porechop #created new env
conda activate porechop
conda install -c bioconda porechop
#!/bin/bash
####### Reserve computing resources #############
#SBATCH --nodes=1
#SBATCH --ntasks=2
#SBATCH --cpus-per-task=4
#SBATCH --time=24:00:00
#SBATCH --mem=15G
#SBATCH --partition=bigmem
####### Run your script #########################
porechop -i passed_reads.fastq.gz -o passed_reads_trimmed.fastq.gz -t 12
sbatch porechop
```
## 2. Filtlong
[Filtlong](https://github.com/rrwick/Filtlong) a tool for filtering long reads. 
You could use job scheduler like SLURM. 
```
conda create --name filtlong -c bioconda filtlong #made new env called filtlong
conda activate filtlong
filtlong --min_mean_q 80 passed_reads_trimmed.fastq.gz| gzip > final_reads.fastq.gz #will produce final_reads.fastq.gz file which will be used for downstream analysis
```
[Mean q](https://github.com/rrwick/Filtlong#read-scoring) is set to 80 to remove the reads that were less than 80% correct which was already done by guppy. So you can play around with different mean_q values to get a better assembly. 
I used 80% and 95% and will compared it using flye. 
95% is too high to get circular genomes. So keep mean q to 80%. 
## 3. Nanoplot
[NanoPlot](https://github.com/wdecoster/NanoPlot) is a good tool to visualize the data
```
module load python/3.10.4
pip3 install NanoPlot
nano nanoplot #creating a job script
#!/bin/bash
####### Reserve computing resources #############
#SBATCH --nodes=1
#SBATCH --ntasks=2
#SBATCH --cpus-per-task=4
#SBATCH --time=24:00:00
#SBATCH --mem=1G
#SBATCH --partition=cpu2019
####### Run your script #########################
NanoPlot -t 8 --fastq final_reads.fastq.gz --maxlength 4000000 --plots dot --legacy hex
sbatch nanoplot_test.sbatch #command to run job script
```
# Assembly
## 1. Flye
[Flye](https://github.com/fenderglass/Flye) is an assembler. Especially good for metagenomic analysis.
```
conda install -c bioconda flye #in filtlong env
conda activate filtlong
nano flye #creating a job script
#!/bin/bash
####### Reserve computing resources #############
#SBATCH --nodes=1
#SBATCH --ntasks=2
#SBATCH --cpus-per-task=4
#SBATCH --time=24:00:00
#SBATCH --mem=15G
#SBATCH --partition=bigmem
####### Run your script #########################
flye --nano-raw final_reads.fastq.gz --meta --genome-size 15m --out-dir assembly_flye -i 0 --threads 8
sbatch flye
```
## 2. Canu
[Canu](https://github.com/marbl/canu/releases) is an older assembly PIPELINE but still works very well. 
All reads are assumed to be raw and untrimmed because it does the trimming and correction
(https://canu.readthedocs.io/en/latest/tutorial.html)
```
curl -L https://github.com/marbl/canu/releases/download/v2.2/canu-2.2.Linux-amd64.tar.xz --output canu-2.2.Linux.tar.xz 
tar -xJf canu-2.2.Linux.tar.xz
nano canu #creating a job script
#!/bin/bash
####### Reserve computing resources #############
#SBATCH --nodes=1
#SBATCH --ntasks=2
#SBATCH --cpus-per-task=4
#SBATCH --time=24:00:00
#SBATCH --mem=15G
#SBATCH --partition=bigmem
####### Run your script #########################
canu -useGrid=remote -p cyano -d cyano genomeSize=4m maxInputCoverage=100 -nanopore passed_reads.fastq.gz
```
## 3. Raven
[Raven](https://github.com/lbcb-sci/raven) is a de novo genome assembler for long uncorrected reads.
```
conda -n raven #created new env 
conda activate raven
conda install -c bioconda raven-assembler
nano raven #creating a job script
#!/bin/bash
####### Reserve computing resources #############
#SBATCH --nodes=1
#SBATCH --ntasks=2
#SBATCH --cpus-per-task=4
#SBATCH --time=24:00:00
#SBATCH --mem=100G
#SBATCH --partition=bigmem
####### Run your script #########################
raven -t 30 -p 4 final_reads.fastq.gz
```
## 4. Trycycler
[Trycycler](https://github.com/rrwick/Trycycler/wiki) is a multiple sequence aligner. It clusters the contif sequences, reconcile the alternative contig sequences, performs MSA, and constructs a consensus sequence from MSA. 
```
conda -n trycyler
conda activate trycycler
conda install trycycler

```
# Polishing
## 1. Minimap2
minimap2 is a mapping software used along side with racon
## 2. Racon
Use it 3 times
## 3. Medaka 
[Medaka](https://github.com/nanoporetech/medaka) a tool to create consensus sequences
Use it 2 times
```
pip install medaka #in filtlong env
conda activate filtlong
nano medaka #creating a job script
#!/bin/bash
####### Reserve computing resources #############
#SBATCH --nodes=1
#SBATCH --ntasks=2
#SBATCH --cpus-per-task=4
#SBATCH --time=24:00:00
#SBATCH --mem=15G
#SBATCH --partition=bigmem
####### Run your script #########################
medaka_consensus -i iassembly.fasta -d test_2.fasta.gz -o medaka_out #-i input_reads.fasta, -d reference.fasta, -o output_directory
sbatch medaka
```
# Bining 
## 1. MetaBAT2
## 2. MaxBin2
## 3. Vamb
