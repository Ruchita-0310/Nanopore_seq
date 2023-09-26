# Nanopore_seq - QC, trimming, and assembly

# Install pod5
```
pip3 install pod5
cp -r /Users/ruchitasolanki/Library/Python/3.9/lib/python/site-packages/pod5 ~/Downloads
```
## Convert .fast5 to .pod5 files
```
cd ~/Downloads/pod5/tools
python3 -m main convert fast5 ~/Downloads/*.fast5 --output ~/Downloads/
```

Transfer .pod5 file from local machine to ARC
```
scp ~/Downloads/output.pod5 ruchita.solanki@arc.ucalgary.ca:/work/ebg_lab/eb/Ruchita_working
```

# Install Dorado
```
tar xf ont-dorado-server_7.1.4_linux64.tar.gz
```

## Find Dorado server config
```
ont-dorado-server/bin/dorado_basecall_server --print_workflows
```
FLO-MIN106     SQK-LSK109                  dna_r9.4.1_450bps_hac          invalid model file

## Start run Dorado on ARC server
```
nano run_dorado.sbatch #to create a script to run it on ARC
#!/bin/bash
####### Reserve computing resources #############
#SBATCH --nodes=1
#SBATCH --ntasks=4
#SBATCH --cpus-per-task=4
#SBATCH --time=048:00:00
#SBATCH --mem=5G
#SBATCH --partition=cpu2019
####### Run your script #########################
ont-dorado-server/bin/dorado_basecall_server --log_path output_folder/server_logs --config dna_r9.4.1_450bps_hac.cfg -p 5555
ls *.pod5 | ont-dorado-server/bin/ont_basecall_client --save_path output_folder/basecall -c dna_r9.4.1_450bps_hac.cfg --port 5555
```
To run the script on ARC
```
sbatch run_dorado.sbatch
```

## To know the status
```
tail -f slurm_file_name.out 
squeue -u ruchita.solanki
watch squeue -u ruchita.solanki
```
# Installing NanoPlot 
NanoPlot produces read length histograms, cumulative yield plots, violin plots of read length and quality over time and bivariate plots comparing the relationship between read
lengths, quality scores, reference identity and read mapping quality.
```
conda install -c bioconda NanoPlot #I used conda
```
