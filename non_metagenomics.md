# Things I tried, but didn't work for my project
# Assembly
## 1. Canu 
- not the best for metagenomics
[Canu](https://github.com/marbl/canu/releases) is an older assembly PIPELINE but still works very well. 
All reads are assumed to be raw and untrimmed because it does the trimming and correction
(https://canu.readthedocs.io/en/latest/tutorial.html)
```
curl -L https://github.com/marbl/canu/releases/download/v2.2/canu-2.2.Linux-amd64.tar.xz --output canu-2.2.Linux.tar.xz 
tar -xJf canu-2.2.Linux.tar.xz
nano canu 
canu -useGrid=remote -p cyano -d cyano genomeSize=4m maxInputCoverage=100 -nanopore passed_reads.fastq.gz
```
## 2. Raven 
- not the best for metagenomics
[Raven](https://github.com/lbcb-sci/raven) is a de novo genome assembler for long uncorrected reads.
```
conda -n raven #created new env 
conda activate raven
conda install -c bioconda raven-assembler
nano raven
####### Run your script #########################
raven -t 30 -p 4 final_reads.fastq.gz
```
## 3. Trycycler 
- not the best for metagenomics
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
- will not work if you have less than 10,000 contigs!
[Vamb](https://github.com/RasmussenLab/vamb)
```
pip install vamb
####### Run your script #########################
vamb --outdir out vamb/ --fasta racon3.fasta --bamfiles sorted.bam -o C 
```
## 2. MaxBin2 
- used metabat2 instead because it gave more bins
- [MaxBin2](https://sourceforge.net/projects/maxbin2/) is a software for binning assembled metagenomic sequences
- copy it on arc in software directory
```
tar -xvf MaxBin-2.2.7
cd MaxBin-2.2.7
cd src
make
cd .. #go back to MaxBin-2.2.7 directory
./autobuild_auxiliary
run_MaxBin.pl -h

####### Run your script #########################
conda activate metawrap-env
run_MaxBin.pl -contig racon3.fasta -out wrap/maxbin_out 
```
- Produced 15 .fasta files/bins and 1 .tar.gz file
## 3. CONCOCT 
- used metabat2 instead because it gave more bins
- [CONCOCT](https://concoct.readthedocs.io/en/latest/usage.html) does unsupervised binning of metagenomic contigs by using nucleotide
composition - kmer frequencies - and coverage data for multiple samples. CONCOCT can accurately (up to species level) bin metagenomic
contigs.
```
conda create -n concoct
conda activate concoct
conda install -c bioconda concoct
#!/bin/bash
####### Reserve computing resources #############
#SBATCH --nodes=1
#SBATCH --ntasks=2
#SBATCH --cpus-per-task=4
#SBATCH --time=24:00:00
#SBATCH --mem=15G
#SBATCH --partition=bigmem
####### Run your script #########################
###cut contigs into smaller parts
cut_up_fasta.py racon3.fasta -c 10000 -o 0 --merge_last -b contigs_10K.bed > contigs_10K.fa
###Generate table with coverage depth information per sample and subcontig
concoct_coverage_table.py contigs_10K.bed sorted.bam > coverage_table.tsv
###Run concoct
concoct --composition_file contigs_10K.fa --coverage_file coverage_table.tsv -b concoct_output/
###Merge subcontig clustering into original contig clustering
merge_cutup_clustering.py concoct_output/clustering_gt1000.csv > concoct_output/clustering_merged.csv
###Extract bins as individual FASTA
mkdir concoct_output/fasta_bins
extract_fasta_bins.py racon3.fasta concoct_output/clustering_merged.csv --output_path concoct_output/fasta_bins
```
## 4. MetaWRAP 
- not need since only one binning tool was used
- [MetaWRAP](https://github.com/bxlab/metaWRAP) 
- You will install metaBAT2 and maxbin2 when you install metawrap. If you want you can skip the above mentioned installation method. 
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
For [bin refinement](https://github.com/bxlab/metaWRAP/blob/master/Usage_tutorial.md) follow step 5
- ```-c 90 -x 5``` the minimum completion is set to 90% and maximum contamination to 5%
- ```-A``` is bins produced by metabat2, ```-B``` is bins produced by maxbin2, and ```-C``` is bins produced by CONCOCT
- Make sure you have nothing other than .fasta or .fa files in your bin directories. 
```
####### Run your script #########################
metawrap bin_refinement -o BIN_REFINEMENT -t 96 -A metabat2_bins/ -B maxbin2_bins/ -C fasta_bins/ -c 90 -x 5 
```
For [reassembly](https://github.com/bxlab/metaWRAP/blob/master/Usage_tutorial.md) follow step 8
```
####### Run your script #########################
metawrap reassemble_bins -o BIN_REASSEMBLY -b metawrap_50_10_bins/ -1 final_reads_1.fastq -2 final_reads_2.fastq --nanopore final_reads.fastq.gz
```
