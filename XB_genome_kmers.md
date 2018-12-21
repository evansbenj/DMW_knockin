# Mapping kmer abundances

I'm using the XB female and male genome data to see if I can find regions of chr8L that have a much higher diversity of kmers in the mother than the father.

I'm working with data from BenF that he trimmed with trimmomatic and scythe.  

First step is to combine the reads:
```
zcat BJE3896_DAD_LOO1_left_paired_scythe.fastq.gz BJE3896_DAD_LOO2_left_paired_scythe.fastq.gz BJE3896_DAD_LOO3_left_paired_scythe.fastq.gz BJE3896_DAD_LOO4_left_paired_scythe.fastq.gz  BJE3896_DAD_LOO5_left_paired_scythe.fastq.gz BJE3896_DAD_LOO6_left_paired_scythe.fastq.gz BJE3896_DAD_LOO7_left_paired_scythe.fastq.gz BJE3896_DAD_LOO8_left_paired_scythe.fastq.gz | gzip -c > BJE3896_DAD_allleft.fastq.gz
```
```
zcat BJE3897_Mom_LOO1_left_paired_scythe.fastq.gz BJE3897_Mom_LOO2_left_paired_scythe.fastq.gz BJE3897_Mom_LOO3_left_paired_scythe.fastq.gz BJE3897_Mom_LOO4_left_paired_scythe.fastq.gz  BJE3897_Mom_LOO5_left_paired_scythe.fastq.gz BJE3897_Mom_LOO6_left_paired_scythe.fastq.gz BJE3897_Mom_LOO7_left_paired_scythe.fastq.gz BJE3897_Mom_LOO8_left_paired_scythe.fastq.gz | gzip -c > BJE3897_Mom_allleft.fastq.gz
```
```
zcat BJE3896_DAD_LOO1_right_paired_scythe.fastq.gz BJE3896_DAD_LOO2_right_paired_scythe.fastq.gz BJE3896_DAD_LOO3_right_paired_scythe.fastq.gz BJE3896_DAD_LOO4_right_paired_scythe.fastq.gz  BJE3896_DAD_LOO5_right_paired_scythe.fastq.gz BJE3896_DAD_LOO6_right_paired_scythe.fastq.gz BJE3896_DAD_LOO7_right_paired_scythe.fastq.gz BJE3896_DAD_LOO8_right_paired_scythe.fastq.gz | gzip -c > BJE3896_DAD_allright.fastq.gz
```
```
zcat BJE3897_Mom_LOO1_right_paired_scythe.fastq.gz BJE3897_Mom_LOO2_right_paired_scythe.fastq.gz BJE3897_Mom_LOO3_right_paired_scythe.fastq.gz BJE3897_Mom_LOO4_right_paired_scythe.fastq.gz  BJE3897_Mom_LOO5_right_paired_scythe.fastq.gz BJE3897_Mom_LOO6_right_paired_scythe.fastq.gz BJE3897_Mom_LOO7_right_paired_scythe.fastq.gz BJE3897_Mom_LOO8_right_paired_scythe.fastq.gz | gzip -c > BJE3897_Mom_allright.fastq.gz
```
