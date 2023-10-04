# Quality control
## 1. Porechop
[Porechop](https://github.com/rrwick/Porechop) eventhough it is no longer available, you could still use
it. 
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
[Filtlong](https://github.com/rrwick/Filtlong) a tool for filtering long reads
You could use job scheduler like SLURM
```
conda create --name filtlong -c bioconda filtlong #made new env called filtlong
conda activate filtlong
filtlong --min_mean_q 80 passed_reads_trimmed.fastq.gz| gzip > chopped_reads.fastq.gz #will produce chopped_reads.fastq.gz file which will be used for downstream analysis
```
[Mean q](https://github.com/rrwick/Filtlong#read-scoring) is set to 80 to remove the reads that were less than 80% correct which was already done by guppy. So you can play around with different mean_q values to get a better assembly. 
I used 80% and 95% and will compared it using flye. 
95% is too high to get circular genomes. So keep mean q to 80%. 
## 3. Nanoplot
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
NanoPlot -t 8 --fastq chopped_reads.fastq.gz --maxlength 4000000 --plots dot --legacy hex
sbatch nanoplot_test.sbatch #command to run job script
```
# Assembly
## 1. Flye
[Flye](https://github.com/fenderglass/Flye) is an assembler. Especially good for metagenomic analysis
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
sbatch flye
```
## 2. Canu
## 3. Raven
## 4. Trycycler
# Polishing
## 1. Minimap2
minimap2 is a mapping software used along side with racon
## 2. Racon

Used 3 time
## 3. Medaka 
[Medaka](https://github.com/nanoporetech/medaka) a tool to create consensus sequences
Use it 2 times
```
pip install medaka #in filtlong env
conda activate filtlong
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