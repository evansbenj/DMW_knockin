# extract chr8L
```
perl -ne 'if(/^>(\S+)/){$c=grep{/^$1$/}qw(Chr8L)}print if $c' Xbo.v1.fa > XB_Austin_Chr8L.fasta
```

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

I just figured out my problem based on this test in the paper describing GenomeTester:  "In this case the difference is calculated with the assumption that the second list is a union containing the first list. Whenever the counts of some k-mer in both lists are equal it means that the first list is the only one among lists in union containing this k-mer and thus it is included in the output list".

So I need to make a union of Mom+Dad before the difference is calculated.  Then using that output list I need to make a union of with each bit of the genome to get a list for each bit that is unique. OK, doable, good to know. So these are the steps:

1. Make union of Mom and Dad kmer lists
2. Subtract Mom kmer list from Mom_Dad_union to get kmers unique to Mom (Mom_unique)
3. Calculate intersection between each chr_bit and Mom_unique (intersection_with_Mom_unique)
4. Calculate proportion of kmers in intersection_with_Mom_unique kmer divided by total number of kmers in each chr_bit 

To compare two kmer databases I made for mom and dad, I first calculated the union of Mom and Dad like this:

```
sbatch run_glistcompare_union.sh ../BJE3896_DAD_kmerlist_31.list ../../Reads_MOM/BJE3897_Mom_kmerlist_31.list Mom_Dad_Union_add
```
where `run_glistcompare_union.sh` is:
```
#!/bin/sh                                                                                            
#SBATCH --job-name=run_glistcompate_union                                                                        
#SBATCH --nodes=4                                                                                    
#SBATCH --ntasks-per-node=32                                                                         
#SBATCH --time=72:00:00                                                                              
#SBATCH --mem=50gb                                                                                   
#SBATCH --output=runglistmaker.out                                                               
#SBATCH --error=runglistmaker.%J.err                                                             
#SBATCH --account=def-ben                                                                            

../../../GenomeTester4/bin/glistcompare $1 $2 --union --rule add -o $3
```
I defined the rule as add even though this is supposed to be the default because the program seems to be doing strange things. After this is done, I need to calculate the difference like this:
```
sbatch run_glistcompare_Mom_minus_Dad.sh
```
where `run_glistcompare_Mom_minus_Dad.sh` is this:
```
#!/bin/sh                                                                                            
#SBATCH --job-name=run_glistmaker                                                                        
#SBATCH --nodes=4                                                                                    
#SBATCH --ntasks-per-node=32                                                                         
#SBATCH --time=72:00:00                                                                              
#SBATCH --mem=50gb                                                                                   
#SBATCH --output=runglistmaker.out                                                               
#SBATCH --error=runglistmaker.%J.err                                                             
#SBATCH --account=def-ben                                                                            

../../../GenomeTester4/bin/glistcompare ../../Reads_MOM/BJE3897_Mom_kmerlist_31.list Mom_Dad_Union_add.list --difference
 --rule first -o Mom_minus_Dad
 ```



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


# Count kmers in each of these lists

I need to count the kmers in each of these lists and then count the kmers in the intersection of each of these lists and the unique kmers from the mom genome.  Then I can plot the proportion of female-specific kmers in each windown of the chr.

```
#!/usr/bin/perl
# This script will run quake on trimmed fq files

my $majorpath = "/home/ben/project/ben/2018_Austin_XB_genome/Trimmed_reads/Reads_DAD/Chr8L_bits/";
opendir (DIR, "/home/ben/project/ben/2018_Austin_XB_genome/Trimmed_reads/Reads_DAD/Chr8L_bits/");



#my @dirs = readdir(DIR);

my @temp;

@files = glob($majorpath."Chr8L*kmer.list_31.list");

# remove files that begin with a dot
foreach my $names ( @files){
    if($names !~ /^[.]/){
	push @temp,$names;
    }
}

#@dirs=@temp;

#print "@dirs";


foreach $file (@files){
    $commandline = "sbatch run_glistquery.sh ".$file." ".$file."_genome.stats";
    print $commandline,"\n";
    $status = system($commandline);
}
```
run_glistquery.sh 
```
#!/bin/sh                                                                                            
#SBATCH --job-name=run_glistquery                                                                        
#SBATCH --nodes=1                                                                                    
#SBATCH --ntasks-per-node=4                                                                         
#SBATCH --time=2:00:00                                                                              
#SBATCH --mem=50gb                                                                                   
#SBATCH --output=runglistquery.out                                                               
#SBATCH --error=runglistquery.%J.err                                                             
#SBATCH --account=def-ben                                                                            

../../../GenomeTester4/bin/glistquery $1 -stat > $2
```

# Calculate intersection

Now I need to get the intersection between the kmer db from each chr bit and the mother-specific kmer db
```
#!/usr/bin/perl
# This script will run quake on trimmed fq files

my $majorpath = "/home/ben/project/ben/2018_Austin_XB_genome/Trimmed_reads/Reads_DAD/Chr8L_bits/";
opendir (DIR, "/home/ben/project/ben/2018_Austin_XB_genome/Trimmed_reads/Reads_DAD/Chr8L_bits/");



#my @dirs = readdir(DIR);

my @temp;

@files = glob($majorpath."Chr8L*kmer.list_31.list");

# remove files that begin with a dot
foreach my $names ( @files){
    if($names !~ /^[.]/){
	push @temp,$names;
    }
}

#@dirs=@temp;

#print "@dirs";


foreach $file (@files){
    $commandline = "sbatch run_glistcompare_intersection.sh ".$file." ".$file."_intersection_list";
    print $commandline,"\n";
    $status = system($commandline);
}
```
and the bash script:
```
#!/bin/sh                                                                                            
#SBATCH --job-name=run_glistmaker                                                                        
#SBATCH --nodes=1                                                                                    
#SBATCH --ntasks-per-node=4                                                                         
#SBATCH --time=0:10:00                                                                              
#SBATCH --mem=50gb                                                                                   
#SBATCH --output=runglistmaker.out                                                               
#SBATCH --error=runglistmaker.%J.err                                                             
#SBATCH --account=def-ben                                                                            

../../../GenomeTester4/bin/glistcompare ../Mom_minus_Dad_31_0_diff1.list $1 --intersection -o $2
```

#Calculate intersection stats

# Summarize the results

Get the number of kmers in each genomic window like this:
```
grep 'NUnique' ../Chr8L_bits/*stats /dev/null > genome_kmers.txt
```

```
grep 'NUnique' ../Chr8L_bits/*list_inters.stats /dev/null > MaternalUnique_intersection_kmers.txt
```

# print out kmers in a list

Finally figured out how to do this:
```
../../../GenomeTester4/bin/glistquery ../Chr8L_bits/Chr8L_29_kmer.list_31.list -f
```

# Meryl

On info in this directory:
```
/2/scratch/ben/2018_Austin_XB_genome/meryl/Linux-amd64/bin
```

Making db with this command:
```
./meryl count ../../../Trimmed_reads/Reads_DAD/BJE3896_DAD_allleft.fastq.gz threads=4 memory=128 k=17 output BJE3896_DAD_allleft_meryl

```
and this one
```
./meryl count ../../../Trimmed_reads/Reads_DAD/BJE3896_DAD_allright.fastq.gz threads=4 memory=128 k=17 output BJE3896_DAD_allright_meryl
```

It was necessary to provide the max memory because the build failed without this after about 12 hours on info (hopefully it works with this).

Now merge the kmer db from both directions
```
./meryl union-sum BJE3897_MOM_allleft_meryl BJE3897_MOM_allright_meryl output BJE3897_MOM_leftright_meryl
```

