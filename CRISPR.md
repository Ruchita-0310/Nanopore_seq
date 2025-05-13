# [Minced](https://github.com/ctSkennerton/minced)
```
conda create -n minced
conda activate minced
#!/bin/bash

# Use a 'for' loop to iterate through the options.
samples=(
  "Baaleninema_simplex_PCC_7105.fna"
  "DL1_bin26.fna"
  "GE22_mb9.fna"
  "Phormidium_sp_Bin_17.fna"
  "Phormidium_sp_S10_Bin050.fna"
  "Sodalinema_B-353.fna"
  "Baaleninema_sp.fna"
  "GE22_mb12.fna"
  "GE7_bin11.fna"
  "Phormidium_sp_CSSed162.fna"
  "Phormidium_sp_SL48-SHIP.fna"
  "sodalinema_sp.fna"
  "Ca_S_alkaliphilum.fna"
  "GE22_mb2.fna"
  "Geitlerinema_sp_BBD_1991_9.fna"
  "Phormidium_spCSSed.fna"
  "Phormidium_willei_BDU_130791.fna"
  "Cyanobacteria_SBC.fna"
  "GE22_mb3.fna"
  "Geitlerinema_sp_P-1104.fna"
  "Phormidium_sp_GEM2_Bin31.fna"
  "Phormidium_yuhuli_AB48.fna"
  "Cyanobacteria_T3Sed10_304.fna"
  "GE22_mb6.fna"
  "Phormidium_lacuna.fna"
  "Phormidium_sp_OSCR.fna"
  "PL4_bin13.fna"
)

# Iterate through all the options.
for sample in "${samples[@]}"; do
  # To get the sample name from ".fna".
  sample_name=$(basename "$sample" .fna)

  # Execute the 'minced' command.
  minced "fna/$sample" "${sample_name}.crisprs" "${sample_name}.gff"

  # This action is performed repeatedly for all options.
done

echo "The 'minced' action has been completed for all options."

```
