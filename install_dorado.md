# Install Dorado
```
tar xf ont-dorado-server_7.1.4_linux64.tar.gz
```

## Find Dorado server config
```
ont-dorado-server/bin/dorado_basecall_server --print_workflows
```
Need this - FLO-MIN106     SQK-LSK109                  dna_r9.4.1_450bps_hac    
## Install pod5
```
pip3 install pod5
```
### Convert .fast5 to .pod5 files
```
cd ~/Downloads/pod5/tools
python3 -m main convert fast5 ~/Downloads/*.fast5 --output ~/Downloads/
```

Transfer .pod5 file from local machine to ARC
```
scp ~/Downloads/output.pod5 arc@server:/path/to/your/directory
```

## Start run Dorado on ARC server
```
nano run_dorado.txt 
####### Run your script #########################
ont-dorado-server/bin/dorado_basecall_server --log_path output_folder/server_logs --config dna_r9.4.1_450bps_hac.cfg -p 5555
ls *.pod5 | ont-dorado-server/bin/ont_basecall_client --save_path output_folder/basecall -c dna_r9.4.1_450bps_hac.cfg --port 5555
sbatch run_dorado.txt
```

## To know the status
```
tail -f slurm_file_name.out
```
