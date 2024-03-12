# Install Dorado
```
tar xf dorado-0.5.3-linux-x64.tar.gz
nano ~/.bashrc
PATH="/home/dorado-0.5.3-linux-x64/bin/:$PATH"
source ~/.bashrc
dorado -h
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
