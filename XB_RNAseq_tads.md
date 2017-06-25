# Trimmming XB RNAseq from tads and adults

First, use a bash script to trim the data
```
#!/bin/bash

files="Xborealis_female_liver
Xborealis_female_tads_Mesonephros
Xborealis_male_liver
Xborealis_male_liver
Xborealis_male_tads_Mesonephros
Xborealis_testis"

for file in $files
do
	java -jar ~/Trimmomatic-0.36/trimmomatic-0.36.jar PE -phred33 -trimlog ${file}_log.txt ${file}_R1.fastq.gz ${file}_R2.fastq.gz ${file}_R1_trim_paired.fastq.gz ${file}_R1_trim_single.fastq.gz ${file}_R2_trim_paired.fastq.gz ${file}_R2_trim_single.fastq.gz ILLUMINACLIP:/home/ben/Trimmomatic-0.36/adapters/TruSeq2-PE_and_XB_tad_adapters.fa:2:30:10 SLIDINGWINDOW:4:15 MINLEN:36
done
```


OK now assemble the data like this:

For only one tissue/sex combination:
```
/home/ben/trinityrnaseq-2.1.1/Trinity --seqType fq --left Xborealis_female_liver_R1_trim_paired.fastq.gz --right Xborealis_female_liver_R2_trim_paired.fastq.gz --CPU 6 --max_memory 40G
```

To make an assembly using both male and female tad data:
```
/home/ben/trinityrnaseq-2.1.1/Trinity --seqType fq --left ../female_tads/Xborealis_female_tads_Mesonephros_R1_trim_paired.fastq.gz,../male_tads/Xborealis_male_tads_Mesonephros_R1_trim_paired.fastq.gz --right ../female_tads/Xborealis_female_tads_Mesonephros_R2_trim_paired.fastq.gz,../male_tads/Xborealis_male_tads_Mesonephros_R2_trim_paired.fastq.gz --CPU 6 --max_memory 40G
```
# Blasting

Make a blast db
```
/usr/local/blast/2.3.0/bin/makeblastdb -in Trinity.fasta -dbtype nucl -out Trinity.fasta_male_tads_blastable
```

and blast in both directions, e.g.:

```
/usr/local/blast/2.3.0/bin/blastn -query /net/infofile4-inside/volume1/scratch/ben/2017_XB_gonads_tads_and_adults/male_tads/trinity_out_dir/male_tads.fasta -db female_tads.fasta_blastable -outfmt 6 -out maletads_to_femaletads.out
```
