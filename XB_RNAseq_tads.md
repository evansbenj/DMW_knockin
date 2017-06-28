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

# Identify transcripts with no hits

I wrote a perl script to identify female tad transcripts with mo hit to the male transcriptome, or vice versa (processes_blast_results_eachsex.pl):

```
#!/usr/bin/perl 

use strict;

######
#
#	This program parses two input files.  One is the fasta query file
#	and the other is the blast results to a fasta database
#
#	The goal is to identify query sequences that have no match to the database
#
#	These are candidate sex-specific loci
#
#####

my %query_hash;
my $key;
my @line;
my @line2;

my $outputfile = "candidate_sex_specific.out";

unless (open(OUTFILE, ">$outputfile"))  {
	print "I can\'t write to $outputfile   $!\n\n";
	exit;
}
print "Creating output file: $outputfile\n";

# First read in the names of the fasta query file

open (DATAINPUT, "/net/infofile4-inside/volume1/scratch/ben/2017_XB_gonads_tads_and_adults/female_tads/trinity_out_dir/female_tads.fasta") or die "Failed to open trop data results";
	while ( my $line = <DATAINPUT>) {
		@line = split (/\s+/,$line);
		@line2 = split (/\>/,$line[0]);
		$query_hash{$line2[1]}=$line[1]; # this should make a hash with key equal to the name of the seq and value equal to "len=XXX"
	}#end while        
close DATAINPUT;

open (DATAINPUT2, "/net/infofile4-inside/volume1/scratch/ben/2017_XB_gonads_tads_and_adults/male_tads/trinity_out_dir/femaletads_to_maletads.out") or die "Failed to open trop data results";
	while ( my $line = <DATAINPUT2>) {
		@line = split (/\s+/,$line);
		$query_hash{$line[0]}=$line[1]; # this should update queries with a hit to equal the name of the seq they match
											# so only those queries with no match will still have a value equal to "len=XXX"						 
	}#end while        
close DATAINPUT2;


foreach $key (sort(keys %query_hash)) {
	# check if the beginning of the value is "len"
	if($query_hash{$key} =~ /^len/){
		print OUTFILE $key,"\t",$query_hash{$key},"\n";
	}	

}

#  close the output file1
close OUTFILE;   
print "Closing output file: $outputfile\n";


exit;

```

Perhaps not surprisingly, there were tons of transcripts from each sex that did not have a match:

```
number male no hits to female tads			36318
number female no hits to male tads			40856
# male transcripts			302389
# female transcripts			303386
proportion male nohits			0.120103575
proportion female nohits			0.134666728
```

So instead I am going to map the data from each transcriptome to a combined assembly and then assess depth of coverage.  My hope is that there will be a smaller set of transcripts that have a very high expression level in females but not males. These can then be explored further.


# Map reads to combined assembly

```

/usr/local/bin/bwa mem -M -t 16 female_and_male_combined.fasta /net/infofile4-inside/volume1/scratch/ben/2017_XB_gonads_tads_and_adults/female_tads/Xborealis_female_tads_Mesonephros_R1_trim_paired.fastq.gz /net/infofile4-inside/volume1/scratch/ben/2017_XB_gonads_tads_and_adults/female_tads/Xborealis_female_tads_Mesonephros_R2_trim_paired.fastq.gz > females_to_combinedtranscriptome.sam

/usr/local/bin/bwa mem -M -t 16 female_and_male_combined.fasta /net/infofile4-inside/volume1/scratch/ben/2017_XB_gonads_tads_and_adults/male_tads/Xborealis_male_tads_Mesonephros_R1_trim_paired.fastq.gz /net/infofile4-inside/volume1/scratch/ben/2017_XB_gonads_tads_and_adults/male_tads/Xborealis_male_tads_Mesonephros_R2_trim_paired.fastq.gz > males_to_combinedtranscriptome.sam
```
make bam files

```
/usr/local/bin/samtools view -bt female_and_male_combined.fasta -o females_to_combinedtranscriptome.bam females_to_combinedtranscriptome.sam

/usr/local/bin/samtools view -bt female_and_male_combined.fasta -o males_to_combinedtranscriptome.bam males_to_combinedtranscriptome.sam
```

delete sam files
```
rm -f *sam
```

sort bam files
```
/usr/local/bin/samtools sort females_to_combinedtranscriptome.bam -o females_to_combinedtranscriptome_sorted.bam

```

make a bai file
```
/usr/local/bin/samtools index females_to_combinedtranscriptome_sorted.bam

```

coverage per site, including scaffolds with no coverage

```
bedtools genomecov -ibam females_to_combinedtranscriptome_sorted.bam -bga > females_to_combinedtranscriptome_sorted_depth_per_site.txt

```
average coverage across all sites in scaffolds with coverage

```
samtools depth  females_to_combinedtranscriptome_sorted.bam  |  awk '{sum+=$3} END { print "Average = ",sum/NR}'
```

genotype
```
~/samtools_2016/bin/samtools mpileup -d8000 -ugf AO248_newtrim_scaffolds.fa -t DP,AD octomys_WGS_to_newgenome_aln_sorted_dedup.bam | ~/samtools_2016/bcftools-1.3.1/bcftools call -V indels --format-fields GQ -m -O z | ~/samtools_2016/bcftools-1.3.1/bcftools filter -e 'FORMAT/GT = "." || FORMAT/DP < 10 || FORMAT/GQ < 20 || FORMAT/GQ = "."' -O z -o oct_WGS_to_newgenome_aln_sorted_dedup.bam.vcf.gz
```

# Get fasta seqs from multifasta file

grep the fill name and then put it in the qw() statement below:
```
perl -ne 'if(/^>(\S+)/){$c=grep{/^$1$/}qw(TRINITY_DN94318_c1_g1_i1 len=358 path=[336:0-357] [-1, 336, -2])}print if $c' female_and_male_combined.fasta > TRINITY_DN94318_c1_g1_i1.fa
```
# mapping transcripts to XL genome

I have made a blast db out of XL v9.1 and blasted the combined tad transcriptome to it, saving the top hit.

To convert the blast output to unique lines based on the first two columns, do this:
```
awk -F"\t" '!seen[$1, $2]++' file > unique.txt

```


### IGNORE BELOW

# Generate list of unique matches 

This can be done by parsing the output based on the first column (-f 1) as defined by a tab delimiter (which is the default, or specify it with =d ' '):
```
cut -f 1 FILENAME | uniq > uniq.txt
```

# Generate a list of query names
```
grep "^>" male_tads.fasta | sed 's/[>]/\t/g' | cut -f 1 -d " "> male_tads_transcript_names.txt
```
and get rid of the tabs:
```
sed -i -e 's/\t//g' male_tads_transcript_names.txt
```


# Compare the files
```
comm -31 uniq.txt male_tads_transcript_names.txt > male_transcripts_with_no_hit_to_female_transcriptome.txt
```
the -31 flag suppresses column 1 which is unique to file 1 and column 3 which are shared by both files

So we end up with names unique to file 2 (i.e. only in the query but not in the match)

