# Things I tried, but didn't work
# Assembly
## 1. Canu - not the best for metagenomics
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
## 2. Raven - not the best for metagenomics
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
## 3. Trycycler - not the best for metagenomics
[Trycycler](https://github.com/rrwick/Trycycler/wiki) is a multiple sequence aligner. 
- It clusters the contif sequences, reconcile the alternative contig sequences, performs MSA, and constructs a consensus sequence from MSA. 
```
conda create -n trycyler
conda activate trycycler
conda install trycycler

#to cluster the assemblies
trycycler cluster --assemblies *.fasta --reads final_reads.fastq.gz --out_dir trycycler_out

#to reconcile the clusters
#make sure to remove all the clusters that have 1 contigs in them
#to print all the good clusters (ie having more than 1 contig in them). You  should have more than 1 cluster that have multiple contigs
#copy all the good clusters to a new directory and run reconcile as a sbatch script. 
for directory in cluster*; do i=$(ls $directory/1_contigs/*.fasta| wc -l); if [ $i -gt 1 ]; then echo $directory; fi; done
trycycler reconcile --reads final_reads.fastq --cluster_dir good_clusters/cluster_001
```
# Binning
## 1. Vamb
[Vamb](https://github.com/RasmussenLab/vamb)
```
pip install vamb
#!/bin/bash
####### Reserve computing resources #############
#SBATCH --nodes=1
#SBATCH --ntasks=2
#SBATCH --cpus-per-task=4
#SBATCH --time=24:00:00
#SBATCH --mem=15G
#SBATCH --partition=bigmem
####### Run your script #########################
vamb --outdir out vamb/ --fasta racon3.fasta --bamfiles sorted.bam -o C 
```
