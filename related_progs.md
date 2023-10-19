# GTDB-TK
```
conda create -n gtdbtk-2.3.2 -c conda-forge -c bioconda gtdbtk=2.3.2
download-db.sh
conda activate gtdbtk-2.3.2
#!/bin/bash
####### Reserve computing resources #############
#SBATCH --nodes=1
#SBATCH --ntasks=2
#SBATCH --cpus-per-task=4
#SBATCH --time=24:00:00
#SBATCH --mem=100G
#SBATCH --partition=bigmem
gtdbtk classify_wf --genome_dir /work/ebg_lab/eb/Ruchita_working/nano_data/passed_qc/assembly_flye_1/wrap/BIN_REFINEMENT/metawrap_50_10_bins --out_dir /work/ebg_lab/eb/Ruchita_working/nano_data/passed_qc/assembly_flye_1/wrap/BIN_REFINEMENT/metawrap_50_10_bins/classify_out --skip_ani_screen --extension fa
```
# CheckM2
```

```
# Samtools
```
conda create -n samtools-env
conda activate samtools-env
conda install -c bioconda samtools
samtools view -Sb sam_file_name.sam -o bam_file_name.bam
samtools sort -o sort.bam bam_file_name.bam --write-index
```
