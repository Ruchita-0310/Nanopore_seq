# Filtering only for guppy basecaller
## 1. Porechop
```
####### Run your script #########################
porechop -i passed_reads.fastq.gz -o passed_reads_trimmed.fastq.gz -t 12
```
## 2. Filtlong
```
####### Run your script #########################
filtlong --min_length 5000 trimmed.fastq > 5_trimmed.fastq
filtlong --min_length 10000 trimmed.fastq > 10_trimmed.fastq
```
# Assembly
## 1. Flye
```
####### Run your script #########################
flye --nano-raw pass_trim.fastq.gz --meta --min-overlap 5000 --out-dir assembly_flye -i 3 --threads 8
```
## 2. Canu
```
####### Run your script #########################
canu-2.2/bin/canu -useGrid=remote -p diatoms -d diatoms -corErrorRate=0.13 -cnsErrorRate=0.25 -nanopore pass_trim.fastq.gz
```
## 3. Miniasm + minimap2
```
####### Run your script ##########
conda activate minisuite
minimap2 -x ava-ont 5_trimmed.fastq 5_trimmed.fastq > ./minimap.paf
miniasm -s 30,000 -m 10,000 -c 5 -d 100,000 -f 5_trimmed.fastq minimap.paf > miniasm.gfa
```
# Polishing
## 1. Racon 3X
```
####### Run your script ##########
####polishing -1
#racon -t 14 ../pass_trim.fastq.gz 1minimap01.sam assembly01.fasta > racon01.fasta
#polishing -2
#racon -t 14 ../pass_trim.fastq.gz 2minimap01.sam racon01.fasta > racon02.fasta
#polishing -3
racon -t 14 ../pass_trim.fastq.gz 3minimap01.sam racon02.fasta > racon03.fasta
```
# 2. Minimap2 3X
```
####### Run your script #########################
#minimap2 -ax map-ont -t 14 assembly01.fasta ../pass_trim.fastq.gz > 1minimap01.sam
#mapping-2
#minimap2 -ax map-ont -t 14 racon01.fasta ../pass_trim.fastq.gz > 2minimap01.sam
#mapping-3
#minimap2 -ax map-ont -t 14 racon02.fasta ../pass_trim.fastq.gz > 3minimap01.sam
```
# 3. Medaka 2X
```
####### Run your script #########################
medaka_consensus -i ../trimmed.fastq -d racon2.fasta -o medaka_out -m r1041_e82_400bps_hac_variant_v4.2.0
##Run 2
medaka_consensus -i ../trimmed.fastq -d consensus.fasta -o medaka2_out -m r1041_e82_400bps_hac_variant_v4.2.0
```

# 4. Proovframe
```
####### Run your script #########################
proovframe map -t 50 -a diatoms_proteins.faa -o proovframe/raw-seqs.tsv racon3.fasta
proovframe fix -o corrected-seqs.fa racon03.fasta raw-seqs.tsv
```
# Bining 
## 1. MetaBAT2
```
####### Run your script #########################
metabat2 -i corrected-seqs.fa -o metabat_out
```
Use ```seqtk``` to extract fasta sequences from bins
```
seqtk sample maxbin_out.003.fasta 
```
To check the number of contigs in each bin
```
grep ">" maxbin_out.002.fasta | wc -l
```
# Seqtk
```
git clone https://github.com/lh3/seqtk.git;
cd seqtk; make
nano name.list #contains all contig numbers you want to extract
../seqtk/seqtk subseq assembly.fasta name.list > diatom.fa
```
# Genome Completeness - for eukaryotes
## 1. BUSCO
```
####### Run your script #########################
conda activate busco
busco -i assembly.fasta -o busco_out -l lineages/eukaryota_odb10 -m genome
busco --download eukaryota_odb10 #to download eukaryote database
busco --download stramenopile_odb10 #to download stramenophile database
```
BUSCO has eukaryotes and stramenophile database. I tried both on metabat_out.1.fa and maxbin_out.010.fasta bins. 
# EukRep-Pipeline 
[Read](https://github.com/patrickwest/EukRep_Pipeline) before using 
1. Classif with EukRep
```
####### Run your script #########################
conda create -y -n eukrep-env -c bioconda scikit-learn==0.19.2 eukrep
conda activate eukrep-env
EukRep -i assembly.fasta -o euk_contigs.fa --min 1000
```
2. Binning - using CONCOCT
```
####### Run your script #########################
conda activate concoct
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
3. Train GeneMark-ES
```
Download GeneMark-ES from http://topaz.gatech.edu/GeneMark/license_download.cgi
tar -xvzf gmes_linux_64_4.tar.gz
conda install bioconda::perl-yaml
conda install bioconda::perl-hash-merge
conda install bioconda::perl-parallel-forkmanager
mamba install perl-mce
cd gmes_linux_64_4
perl gmes_petap.pl
```
# Telomere Identification
```
conda install -c bioconda tidk
tidk explore --minimum 1 --maximum 20 euk.fasta
tidk explore -l -m 1 -x 200 diatoms.fa
tidk find --clade Chlamydomonadales --output find --dir ./  euk.fasta
tidk search --string AAAAAG --output search --dir ./ euk.fasta
tidk plot --tsv find_telomeric_repeat_windows.tsv
```
## Results
Exploring genome for potential telomeric repeats between lengths 5 and 12. Canonical_repeat_unit	= AAAAAG	and count = 594
# Annotation
## 1. MetaErg 
```
####### Run your script #########################
\time  singularity exec --bind /work/ebg_lab/referenceDatabases/metaerg_db_V214:/databases --bind /home/ruchita.solanki/dorado/flye_out:/data  --writable /work/ebg_lab/software/metaerg-v2.5.2/sandbox_metaerg_2.5.4/ metaerg -- database_dir /databases --contig_file /data --file_extension .fasta
```
## 2. Barrnap
```
####### Run your script #########################
barrnap /work/ebg_lab/eb/Ruchita_working/guppy/LIB/assembly.fasta --kingdom euk --outseq /work/ebg_lab/eb/Ruchita_working/guppy/LIB/euk
barrnap /work/ebg_lab/eb/Ruchita_working/guppy/LIB/assembly.fasta --kingdom mito --outseq /work/ebg_lab/eb/Ruchita_working/guppy/LIB/mito
```
