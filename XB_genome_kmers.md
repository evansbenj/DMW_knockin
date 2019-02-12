# Mapping kmer abundances

I'm using the XB female and male genome data to see if I can find regions of chr8L that have a much higher diversity of kmers in the mother than the father.

Directory:
```
/work/ben/2018_Austin_XB_genome/
```

Prepare the XB genome:
```
bwa index -a bwtsw AO248_newtrim_scaffolds.fa
samtools faidx Xbo.v1.fa
~/jre1.8.0_111/bin/java -Xmx2g -jar ~/picard-tools-1.131/picard.jar CreateSequenceDictionary REFERENCE=Xbo.v1.fa OUTPUT=Xbo.v1.dict
```

Combine the reads from BenF that he trimmed with trimmomatic and scythe:
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
# GenomeTester

I am working in this directory on graham:
```
/home/ben/project/ben/2018_Austin_XB_genome/Trimmed_reads/Reads_DAD
```

To compare two kmer databases I made for mom and dad, I ran this command:
```
sbatch run_glistcompare.sh
```

This made a 50Gb file called 'Mom_minus_Dad_31_0_diff1.list'.  I used this 'run_glistquery.sh' to get this information about this kmer list file:
```
Wordlength	31
NUnique	4462866441
NTotal	29602576276
```

I'm trying to instead use a cutoff of 10 (so only kmers that show up 10 times in Mom and never in Dad are kept), just to see if this dramatically changes the file size.
