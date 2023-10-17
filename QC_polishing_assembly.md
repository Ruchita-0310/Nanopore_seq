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
## 1. Flye - works well for metagenomics 
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
[Seqtk](https://github.com/lh3/seqtk) is a tool kit that helps you create a subset of sequences to run checkm2/gtdb-tk on.
While trycycler is running, you could download [Bandage](https://github.com/rrwick/Bandage) to visualize the contigs produced by Flye. Some contigs will be circular and you could run checkm and gtdb-tk. 
```
conda install -c bioconda seqtk
nano seqtk_names
seqtk subseq assembly.fasta seqtk_names > out.fq
```
## 2. Canu - not the best for metagenomics
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
## 3. Raven - not the best for metagenomics
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
## 4. Trycycler - not the best for metagenomics
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
# Polishing
## 1. Minimap2 
[Minimap2](https://github.com/lh3/minimap2) is a versatile sequence alignment program that aligns DNA or mRNA sequences against a large
reference database. 
- It also finds overlaps between long reads with error rate up to ~15%.
- Used along side with Racon 3 times
```
conda create -n minisuite
conda activate minisuite
conda install -c bioconda minimap2
#!/bin/bash
####### Reserve computing resources #############
#SBATCH --nodes=1
#SBATCH --ntasks=2
#SBATCH --cpus-per-task=4
#SBATCH --time=24:00:00
#SBATCH --mem=100G
#SBATCH --partition=bigmem
conda activate minisuite
###mapping -1
minimap2 -ax map-ont -t 14 assembly.fasta final_reads.fastq.gz > /work/ebg_lab/eb/Ruchita_working/nano_data/passed_qc/assembly_flye_1/minimap2.sam
###mapping -2
minimap2 -ax map-ont -t 14 racon1.fasta final_reads.fastq.gz > /work/ebg_lab/eb/Ruchita_working/nano_data/passed_qc/assembly_flye_1/2minimap2.sam
###mapping -3
minimap2 -ax map-ont -t 14 racon2.fasta final_reads.fastq.gz > /work/ebg_lab/eb/Ruchita_working/nano_data/passed_qc/assembly_flye_1/3minimap2.sam
###mapping -4
minimap2 -ax map-ont -t 14 racon3.fasta final_reads.fastq.gz > /work/ebg_lab/eb/Ruchita_working/nano_data/passed_qc/assembly_flye_1/3minimap3.sam

###create .bam file of 3minimap3.sam.
```
## 2. Racon
[Racon](https://github.com/isovic/racon) is a standalone consensus module to correct raw contigs generated by rapid assembly methods which
do not include a consensus step. 
```
conda create -n racon
conda activate racon
conda install -c bioconda racon
#!/bin/bash
####### Reserve computing resources #############
#SBATCH --nodes=1
#SBATCH --ntasks=2
#SBATCH --cpus-per-task=4
#SBATCH --time=24:00:00
#SBATCH --mem=100G
#SBATCH --partition=bigmem
conda activate racon
###polishing -1 
racon -t 14 assembly.fasta minimap2.sam final_reads.fastq.gz > racon1.fasta
###polishing -2
racon -t 14 assembly.fasta 2minimap2.sam racon1.fasta > racon2.fasta
###polishing -3
racon -t 14 assembly.fasta 3minimap2.sam racon2.fasta > racon3.fasta
```
## 3. Medaka 
[Medaka](https://github.com/nanoporetech/medaka) a tool to create consensus sequences
Use it 2 times
```
conda create -n py37env python=3.7
conda activate py37env
python --version #Python 3.7.16
pip install medaka
pip install numpy==1.21.6
pip install tensorflow==2.10.0    
conda install -c bioconda bcftools=1.15.1
conda install -c bioconda bgzip     
conda install -c bioconda minimap2 
conda install -c bioconda samtools  
conda install -c bioconda tabix=1.11  
#!/bin/bash
####### Reserve computing resources #############
#SBATCH --nodes=1
#SBATCH --ntasks=2
#SBATCH --cpus-per-task=4
#SBATCH --time=24:00:00
#SBATCH --mem=15G
#SBATCH --partition=bigmem
####### Run your script #########################
conda activate py37env
medaka_consensus -i final_reads.fastq.gz -d racon3.fasta -o medaka_out #-i input_reads.fasta, -d reference.fasta, -o output_directory
sbatch medaka
```
# Bining 
## 1. MetaBAT2
[MetaBAT2](https://bitbucket.org/berkeleylab/metabat/src/master/)
```
wget https://bitbucket.org/berkeleylab/metabat/get/master.tar.gz
tar xzvf master.tar.gz
cd berkeleylab-metabat-*
mkdir build && cd build && cmake /your/path/build
#!/bin/bash
####### Reserve computing resources #############
#SBATCH --nodes=1
#SBATCH --ntasks=2
#SBATCH --cpus-per-task=4
#SBATCH --time=24:00:00
#SBATCH --mem=15G
#SBATCH --partition=bigmem
####### Run your script #########################
metabat2 -i racon3.fasta -o metabat_out -s 500000
```
## 2. MaxBin2
[MaxBin2](https://sourceforge.net/projects/maxbin2/)
- copy it on arc in software directory
```
tar -xvf MaxBin-2.2.7
cd MaxBin-2.2.7
cd src
make
cd .. #go back to MaxBin-2.2.7 directory
./autobuild_auxiliary
run_MaxBin.pl -h
#!/bin/bash
####### Reserve computing resources #############
#SBATCH --nodes=1
#SBATCH --ntasks=2
#SBATCH --cpus-per-task=4
#SBATCH --time=24:00:00
#SBATCH --mem=15G
#SBATCH --partition=bigmem
####### Run your script #########################
run_MaxBin.pl -contig racon3.fasta -out maxbin_out 
```
## 3. Vamb
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
## 4. MetaWRAP
- You will install metaBAT2 and maxbin2 when you install metawrap. If you want you can skip the previous installation method. 
```
mamba create -y -n metawrap-env python=2.7
mamba activate metawrap-env
git clone https://github.com/bxlab/metaWRAP.git
mkdir MY_CHECKM_FOLDER
cd MY_CHECKM_FOLDER
wget https://data.ace.uq.edu.au/public/CheckM_databases/checkm_data_2015_01_16.tar.gz
tar -xvf *.tar.gz
rm *.gz
cd ../
mamba install biopython blas=2.5 blast=2.6.0 bmtagger bowtie2 bwa checkm-genome fastqc krona=2.7 matplotlib maxbin2 megahit metabat2 pandas prokka quast r-ggplot2 r-recommended salmon samtools=1.9 seaborn spades trim-galore concoct=1.0 pplacer

```
