# Install Dorado
```
tar xf dorado-0.5.3-linux-x64.tar.gz
nano ~/.bashrc
PATH="/home/dorado-0.5.3-linux-x64/bin/:$PATH"
source ~/.bashrc
dorado -h
```
# Basecalling models
Read about which model to use before basecalling
https://github.com/nanoporetech/dorado
```
dorado download --model all
nano basecalling.txt
#!/bin/bash
####### Reserve computing resources #############
#SBATCH --nodes=1
#SBATCH --ntasks=2
#SBATCH --cpus-per-task=4
#SBATCH --time=24:00:00
#SBATCH --mem=200G
#SBATCH --partition=bigmem
####### Run your script #########################
#!/bin/bash
dorado basecaller dna_r10.4.1_e8.2_400bps_hac@v4.3.0 /path/to/pod5/files/ -x cpu --kit-name SQK-NBD1$ 
sbatch bascalling.txt
```
`dna_r10.4.1_e8.2_400bps_hac@v4.3.0` the one I used.
You can run it on GPU as well as CPU. 


