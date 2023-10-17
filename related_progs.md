# Installing GTDB-TK
```
conda create -n gtdbtk-2.3.2 -c conda-forge -c bioconda gtdbtk=2.3.2
download-db.sh
gtdbtk classify_wf --genome_dir /work/ebg_lab/eb/Ruchita_working/nano_data/passed_qc/trycycler/gtdbtk --out_dir /work/ebg_lab/eb/Ruchita_working/nano_data/passed_qc/trycycler/g_out --extension gz --cpus 2
```
# Installing CheckM2
```

```
# Installing samtools
```
wget https://github.com/samtools/samtools/releases/download/1.18/samtools-1.18.tar.bz2
tar -xvf samtools-1.18.tar.bz2
cd samtools-1.18/
./configure


conda create -n samtools-env
conda activate samtools-env
conda install -c bioconda samtools
samtools view -Sb 3minimap2.sam -o 3minimap2.bam
samtools sort -o sorted.bam 3minimap2.bam
```
