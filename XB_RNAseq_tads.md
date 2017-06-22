# Trimmming XB RNAseq from tads and adults

First, use a bash script to trim the data
```
#!/bin/bash

files=”Xborealis_female_liver
Xborealis_female_tads_Mesonephros
Xborealis_male_liver
Xborealis_male_liver
Xborealis_male_tads_Mesonephros
Xborealis_testis”

for file in $files
do
	java -jar ~/Trimmomatic-0.36/trimmomatic-0.36.jar PE -phred33 -trimlog ${file}_log.txt ${file}_R1.fastq.gz ${file}_R2.fastq.gz ${file}_R1_trim_paired.fastq.gz ${file}_R1_trim_single.fastq.gz ${file}_R2_trim_paired.fastq.gz ${file}_R2_trim_single.fastq.gz ILLUMINACLIP:~/Trimmomatic-0.36/adapters/TruSeq2-PE_and_XB_tad_adapters.fa:2:30:10 SLIDINGWINDOW:4:15 MINLEN:36
done
```