# Installing GTDB-TK
```
conda create -n gtdbtk-2.3.2 -c conda-forge -c bioconda gtdbtk=2.3.2
download-db.sh
gtdbtk classify_wf --genome_dir /work/ebg_lab/eb/Ruchita_working/nano_data/passed_qc/trycycler/gtdbtk --out_dir /work/ebg_lab/eb/Ruchita_working/nano_data/passed_qc/trycycler/g_out --extension gz --cpus 2
```
