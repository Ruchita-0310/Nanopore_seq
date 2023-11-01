# Samtools
It will be important for you to convert .sam files to .bam files and indexing .bam files. 
```
conda create -n samtools-env
conda activate samtools-env
conda install -c bioconda samtools
samtools view -Sb sam_file_name.sam -o bam_file_name.bam
samtools sort -o sort.bam bam_file_name.bam --write-index
```
# Proovframe
- You will need [Diamond](https://github.com/bbuchfink/diamond/wiki) 
```
mkdir diamond-dir
mv diamond-linux64.tar.gz diamond-dir/
cd diamond-dir
tar xzf diamond-linux64.tar.gz
export PATH="$PATH:/home/ruchita.solanki/diamond-dir"
echo $PATH

export PATH="$PATH:$/home/ruchita.solanki/proovframe/bin"
echo 'export PATH="$PATH:/home/ruchita.solanki/proovframe/bin"' >> ~/.bashrc
source ~/.bashrc
#!/bin/bash
####### Reserve computing resources #############
#SBATCH --nodes=1
#SBATCH --ntasks=2
#SBATCH --cpus-per-task=4
#SBATCH --time=24:00:00
#SBATCH --mem=15G
#SBATCH --partition=bigmem
####### Run your script #########################
proovframe map -t 50 -a all_proteins.faa -o proovframe/raw-seqs.tsv racon3.fasta

