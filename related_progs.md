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
conda create -n samtools-env
conda activate samtools-env
conda install -c bioconda samtools
samtools view -Sb sam_file_name.sam -o bam_file_name.bam
samtools sort -o sorted_bam_file_name.bam bam_file_name.bam
```
