# Installing and running softwares on ARC using conda and pip
## Filtlong
```
conda create --name filtlong -c bioconda filtlong #made new env called filtlong
conda activate filtlong
```
To run filtlong
```
filtlong --min_mean_q 9 passed_reads.fastq.gz| gzip > test.fastq.gz #will produce test.fastq.gz file which will be used for nanoplot
```
Setting min_mean_q to 7 ensures that reads with an average quality score below 7 will be filtered out. This helps remove reads that are more
likely to contain sequencing errors or low-quality data, which can negatively impact downstream analyses such as genome assembly or variant
calling.
## Nanoplot
```
module load python/3.10.4
pip3 install NanoPlot
nano nanoplot_test.sbatch #creating a job script
#!/bin/bash
####### Reserve computing resources #############
#SBATCH --nodes=1
#SBATCH --ntasks=2
#SBATCH --cpus-per-task=4
#SBATCH --time=24:00:00
#SBATCH --mem=1G
#SBATCH --partition=cpu2019
####### Run your script #########################
NanoPlot -t 8 --fastq test.fastq.gz --maxlength 4000000 --plots dot --legacy hex
sbatch nanoplot_test.sbatch #command to run job script
```
## Flye
```
conda install -c bioconda flye #in filtlong env
#!/bin/bash
####### Reserve computing resources #############
#SBATCH --nodes=1
#SBATCH --ntasks=2
#SBATCH --cpus-per-task=4
#SBATCH --time=72:00:00
#SBATCH --mem=15G
#SBATCH --partition=cpu2019
####### Run your script #########################
flye --nano-raw test.fastq.gz --meta --genome-size 50m --out-dir assembly_flye -i 0 --threads 8


##ERROR
[2023-09-27 21:06:59] ERROR: Looks like the system ran out of memory
[2023-09-27 21:06:59] ERROR: Command '['flye-modules', 'assemble', '--reads', '/work/ebg_lab/eb/Ruchita_working/nano_data/test_qc/test.fastq.gz', '--out-asm', '/work/ebg_lab/eb/Ruchita_working/nano_data/test_qc/assembly_flye/00-assembly/draft_assembly.fasta', '--config', '/home/ruchita.solanki/.conda/envs/filtlong/lib/python3.10/site-packages/flye/config/bin_cfg/asm_raw_reads.cfg', '--log', '/work/ebg_lab/eb/Ruchita_working/nano_data/test_qc/assembly_flye/flye.log', '--threads', '4', '--meta', '--genome-size', '450000000', '--min-ovlp', '3000']' died with <Signals.SIGKILL: 9>.
```
## Medaka
```
pip install medaka #in filtlong env
```
