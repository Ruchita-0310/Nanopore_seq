# Nanopore_seq

Install pod5
```
pip3 install pod5
cp -r /Users/ruchitasolanki/Library/Python/3.9/lib/python/site-packages/pod5 ~/Downloads
```

Convert .fast5 to .pod5 files
```
cd ~/Downloads/pod5/tools
python3 -m main convert fast5 ~/Downloads/*.fast5 --output ~/Downloads/
```

Transfer .pod5 file from local machine to ARC
```
scp ~/Downloads/output.pod5 ruchita.solanki@arc.ucalgary.ca:/work/ebg_lab/eb/Ruchita_working
```

Install Dorado
```
tar xf ont-dorado-server_7.1.4_linux64.tar.gz
```

Find Dorado server config
```
ont-dorado-server/bin/dorado_basecall_server --print_workflows
```
FLO-MIN106     SQK-LSK109                  dna_r9.4.1_450bps_hac          invalid model file

Start run Dorado on ARC server
```
nano run_dorado.sbatch #to create a script to run it on ARC
#!/bin/bash
####### Reserve computing resources #############
#SBATCH --nodes=1
#SBATCH --ntasks=2
#SBATCH --cpus-per-task=1
#SBATCH --time=01:00:00
#SBATCH --mem=1G
#SBATCH --partition=cpu2019
####### Run your script #########################
ont-dorado-server/bin/dorado_basecall_server --log_path output_folder/server_logs --config dna_r9.4.1_450bps_hac.cfg -p 5555
ls *.pod5 | ont-dorado-server/bin/ont_basecall_client --save_path output_folder/basecall -c dna_r9.4.1_450bps_hac.cfg --port 5555
```
To know the status
```
tail -f slurm-23027071.out 
squeue -u ruchita.solanki
watch squeue -u ruchita.solanki
```
