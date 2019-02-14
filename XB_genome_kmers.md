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

# Split Fasta file

I wrote a script to split up fasta files:
```
#!/usr/bin/env perl
use strict;
use warnings;
use List::MoreUtils qw/ uniq /;



#  This program reads in a fasta file and splits it into several
# smaller files of length length.  This is useful for doing knmer quantification.
# Output files will be printed to "output_directory"



# run it like this
# Split_fasta.pl input.fasta length output_directory 


my $inputfile = $ARGV[0];
my $length = $ARGV[1]; # this how many lines each subset file will have
my $outputdirectory = $ARGV[2];

unless (open DATAINPUT, $inputfile) {
	print 'Can not find the input file.\n';
	exit;
}


my $filecounter=0;
my $linecounter=0;
my $header;
my @temp;
my $outputfile;
my $line;

# Read in datainput file

while ( my $line = <DATAINPUT>) {
	if($line =~ '>'){
		# print HEADER to output file
		$filecounter+=1;
		# parse header
		chomp($line);
		@temp=split('>',$line);
		$header=$temp[1];
		unless (open(OUTFILE, ">".$outputdirectory."\/".$header."_".$filecounter))  {
			print "I can\'t write to $outputfile\n";
			exit;
		}
		print OUTFILE ">".$header."_".$filecounter."\n";
		$linecounter=0;
	}
	elsif($linecounter < $length){
		print OUTFILE $line;
		$linecounter+=1;
	}	
	else{	
		$filecounter+=1;
		unless (open(OUTFILE, ">".$outputdirectory."\/".$header."_".$filecounter))  {
			print "I can\'t write to $outputfile\n";
			exit;
		}
		print OUTFILE ">".$header."_".$filecounter."\n";
		print OUTFILE $line;
		$linecounter=0;
	}
}		

```

# Count kmers in each fasta file

In this directory on graham:
```
/home/ben/project/ben/2018_Austin_XB_genome/Trimmed_reads/Reads_DAD/ben_scripts
```
I used this script to launch kmer counting of all the chr7 bits:
```
#!/usr/bin/perl
# This script will run quake on trimmed fq files

my $majorpath = "/home/ben/project/ben/2018_Austin_XB_genome/Trimmed_reads/Reads_DAD/Chr8L_bits/";
opendir (DIR, "/home/ben/project/ben/2018_Austin_XB_genome/Trimmed_reads/Reads_DAD/Chr8L_bits/");



#my @dirs = readdir(DIR);

my @temp;

@files = glob($majorpath."Chr8L*");

# remove files that begin with a dot
foreach my $names ( @files){
    if($names !~ /^[.]/){
	push @temp,$names;
    }
}

#@dirs=@temp;

#print "@dirs";


foreach $file (@files){
    $commandline = "sbatch run_glistmaker.sh ".$file." ".$file."_kmer.list";
    print $commandline,"\n";
    $status = system($commandline);
}
```
and this exectutes this bash script (run_glistmaker.sh):
```
#!/bin/sh                                                                                            
#SBATCH --job-name=run_glistmaker                                                                        
#SBATCH --nodes=1                                                                                    
#SBATCH --ntasks-per-node=12                                                                         
#SBATCH --time=1:00:00                                                                              
#SBATCH --mem=50gb                                                                                   
#SBATCH --output=runglistmaker.%J.out                                                               
#SBATCH --error=runglistmaker.%J.err                                                             
#SBATCH --account=def-ben                                                                            

../../../GenomeTester4/bin/glistmaker $1 -w 31 -o $2
```



