# 1. Quality control
Before doing any quality control, concatenate all the passed reads into one file 
```
cat ./fastq_pass/*.fastq.gz > passed_reads.fastq.gz
```
## 1.1 Porechop
[Porechop](https://github.com/rrwick/Porechop) eventhough it is no longer available, you could still use
it. 
Guppy basecaller does the adapter trimming. Still better to use porechop. 
```
conda create -n porechop #created new env
conda activate porechop
conda install -c bioconda porechop
####### Run your script #########################
porechop -i passed_reads.fastq.gz -o passed_reads_trimmed.fastq.gz -t 12
sbatch porechop
```
## 1.2 Filtlong
[Filtlong](https://github.com/rrwick/Filtlong) a tool for filtering long reads. 
You could use job scheduler like SLURM. 
```
conda create --name filtlong -c bioconda filtlong #made new env called filtlong
conda activate filtlong
filtlong --min_mean_q 80 passed_reads_trimmed.fastq.gz| gzip > final_reads.fastq.gz
#will produce final_reads.fastq.gz file which will be used for downstream analysis
```
[Mean q](https://github.com/rrwick/Filtlong#read-scoring) is set to 80 to remove the reads that were less than 80% correct which was already done by guppy. So you can play around with different mean_q values to get a better assembly. 
I used 80% and 95% and will compared it using flye. 
95% is too high to get circular genomes. So keep mean q to 80%. 
## 1.3 Nanoplot
[NanoPlot](https://github.com/wdecoster/NanoPlot) is a good tool to visualize the data
```
module load python/3.10.4
pip3 install NanoPlot
nano nanoplot #creating a job script
####### Run your script #########################
NanoPlot -t 8 --fastq final_reads.fastq.gz --maxlength 4000000 --plots dot --legacy hex
sbatch nanoplot_test.sbatch
```
# 2. Assembly
## 2.1 Flye - works well for metagenomics 
[Flye](https://github.com/fenderglass/Flye) is an assembler. Especially good for metagenomic analysis.
```
conda install -c bioconda flye #in filtlong env
conda activate filtlong
nano flye #creating a job script
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
# 3. Polishing
## 3.1 Minimap2 
[Minimap2](https://github.com/lh3/minimap2) is a versatile sequence alignment program that aligns DNA or mRNA sequences against a large
reference database. 
- It also finds overlaps between long reads with error rate up to ~15%.
- Used along side with Racon 3 times
```
conda create -n minisuite
conda activate minisuite
conda install -c bioconda minimap2
####### Run your script #########################
###mapping -1
minimap2 -ax map-ont -t 14 assembly.fasta final_reads.fastq.gz > 1minimap01.sam
###mapping -2
minimap2 -ax map-ont -t 14 racon1.fasta final_reads.fastq.gz > 2minimap01.sam
###mapping -3
minimap2 -ax map-ont -t 14 racon2.fasta final_reads.fastq.gz > 3minimap01.sam
```
## 3.2 Racon
[Racon](https://github.com/isovic/racon) is a standalone consensus module to correct raw contigs generated by rapid assembly methods which
do not include a consensus step. 
```
conda create -n racon
conda activate racon
conda install -c bioconda racon
####### Run your script #########################
#polishing -1 
racon -t 14 t01.fastq 1minimap01.sam assembly01.fasta > racon01.fasta
#polishing -2
racon -t 14 racon01.fasta 2minimap2.sam assembly01.fasta > racon02.fasta
#polishing -3
racon -t 14 racon02.fasta 3minimap2.sam assembly01.fasta > racon03.fasta
```
## 3.3 Proovframe
- [Proovframe](https://github.com/thackl/proovframe) detects and corrects frameshifts in coding sequences from raw long reads or long-read
derived assemblies.
- You will need [Diamond](https://github.com/bbuchfink/diamond/wiki) 
```
#diamond
mkdir diamond-dir
mv diamond-linux64.tar.gz diamond-dir/
cd diamond-dir
tar xzf diamond-linux64.tar.gz
export PATH="$PATH:/path/to/diamond-dir"
echo $PATH

#proovframe
git clone https://github.com/thackl/proovframe
export PATH="$PATH:$/path/to/proovframe/bin"
echo 'export PATH="$PATH:/path/to/proovframe/bin"' >> ~/.bashrc
source ~/.bashrc
# map proteins to reads
# make directory named proovframe
proovframe map -t 50 -a all_proteins.faa -o proovframe/raw-seqs.tsv racon3.fasta
# fix frameshifts in reads
# run it in proovframe directory
proovframe fix -o corrected-seqs.fa racon3.fasta raw-seqs.tsv
```
## 3.4 Medaka 
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
####### Run your script #########################
##Run 1
medaka_consensus -i final_reads.fastq.gz -d racon3.fasta -o medaka_out -m r941_e81_fast_g514
##Run 2
medaka_consensus -i final_reads.fastq.gz -d consensus.fasta -o medaka2_out -m r941_e81_fast_g514
```
# 4. Bining 
## 4.1 MetaBAT2
[MetaBAT2](https://bitbucket.org/berkeleylab/metabat/src/master/) is statistical framework for reconstructing genomes from metagenomic data. 
```
wget https://bitbucket.org/berkeleylab/metabat/get/master.tar.gz
tar xzvf master.tar.gz
cd berkeleylab-metabat-*
mkdir build && cd build && cmake /your/path/build && make && make test && make install
####### Run your script #########################
conda activate metawrap-env
metabat2 -i racon3.fasta -o wrap/metabat_out -s 500000
```
- Produced 38 .fa files/bins 
# 5. Assembly processing 
## 5.1 CheckM2
[CheckM2](https://github.com/chklovski/CheckM2)
```
mamba create -n checkm2 -c bioconda -c conda-forge checkm2
mamba activate checkm2
mamba install python=3.8
checkm2 database --download
####### Run your script #########################
conda activate checkm2
checkm2 predict -t 30 -x fa --input ./ --output-directory ./CheckM2 
```
## 5.2 GTDB-Tk
[GTDB-Tk](https://github.com/Ecogenomics/GTDBTk)
```
conda create -n gtdbtk-2.3.2 -c conda-forge -c bioconda gtdbtk=2.3.2
download-db.sh
conda activate gtdbtk-2.3.2
####### Run your script #########################
gtdbtk classify_wf --genome_dir /path/to/metawrap_50_10_bins --out_dir /path/to/output/directory --skip_ani_screen --extension fa
```
## 5.3 Prodigal
[Prodigal](https://github.com/hyattpd/prodigal/wiki)
