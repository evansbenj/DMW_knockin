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
and
```
./meryl union-sum BJE3896_DAD_allleft_meryl BJE3896_DAD_allright_meryl output BJE3896_DAD_leftright_meryl
```

Now subtract dad from mom to get unique kmers in mom

```
./meryl difference BJE3897_MOM_leftright_meryl BJE3896_DAD_leftright_meryl output MOM_unique
```
and as a control do the same for dad
```
./meryl difference BJE3896_DAD_leftright_meryl BJE3897_MOM_leftright_meryl output DAD_unique
```
Now make kmer dbs for each section of chr8L
	* I did this on graham using this script : 'sbatch_commando_makes_kmer_meryl_lists.pl'
	```
	#!/usr/bin/perl
	# This script will run quake on trimmed fq files

	my $majorpath = "/home/ben/project/ben/2018_Austin_XB_genome/Trimmed_reads/Reads_DAD/XL_v9.1_chr8L_bits/";
	opendir (DIR, "/home/ben/project/ben/2018_Austin_XB_genome/Trimmed_reads/Reads_DAD/XL_v9.1_chr8L_bits/");



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
	    $commandline = "sbatch run_meryl_listmaker.sh ".$file." ".$file."_kmer.list";
	    print $commandline,"\n";
	    $status = system($commandline);
	}
	```

	* in this directory: '/home/ben/project/ben/2018_Austin_XB_genome/Trimmed_reads/Reads_DAD/ben_scripts'

or on info using a bash command:

```
for file in *
do
  ../meryl count "$file" threads=4 memory=125 k=17 output "$file"_meryllist
done

```
or, more laborously:
```
../meryl count Chr8L_24 threads=4 memory=125 k=17 output Chr8L_24_meryllist
../meryl count Chr8L_25 threads=4 memory=125 k=17 output Chr8L_25_meryllist
../meryl count Chr8L_26 threads=4 memory=125 k=17 output Chr8L_26_meryllist
../meryl count Chr8L_27 threads=4 memory=125 k=17 output Chr8L_27_meryllist
../meryl count Chr8L_28 threads=4 memory=125 k=17 output Chr8L_28_meryllist
../meryl count Chr8L_29 threads=4 memory=125 k=17 output Chr8L_29_meryllist
../meryl count Chr8L_30 threads=4 memory=125 k=17 output Chr8L_30_meryllist
../meryl count Chr8L_1 threads=4 memory=125 k=17 output Chr8L_1_meryllist
../meryl count Chr8L_2 threads=4 memory=125 k=17 output Chr8L_2_meryllist
../meryl count Chr8L_3 threads=4 memory=125 k=17 output Chr8L_3_meryllist
../meryl count Chr8L_4 threads=4 memory=125 k=17 output Chr8L_4_meryllist
../meryl count Chr8L_5 threads=4 memory=125 k=17 output Chr8L_5_meryllist
../meryl count Chr8L_6 threads=4 memory=125 k=17 output Chr8L_6_meryllist
../meryl count Chr8L_7 threads=4 memory=125 k=17 output Chr8L_7_meryllist
../meryl count Chr8L_8 threads=4 memory=125 k=17 output Chr8L_8_meryllist
../meryl count Chr8L_9 threads=4 memory=125 k=17 output Chr8L_9_meryllist
../meryl count Chr8L_10 threads=4 memory=125 k=17 output Chr8L_10_meryllist
../meryl count Chr8L_11 threads=4 memory=125 k=17 output Chr8L_11_meryllist
../meryl count Chr8L_12 threads=4 memory=125 k=17 output Chr8L_12_meryllist
../meryl count Chr8L_13 threads=4 memory=125 k=17 output Chr8L_13_meryllist
../meryl count Chr8L_14 threads=4 memory=125 k=17 output Chr8L_14_meryllist
../meryl count Chr8L_15 threads=4 memory=125 k=17 output Chr8L_15_meryllist
../meryl count Chr8L_16 threads=4 memory=125 k=17 output Chr8L_16_meryllist
../meryl count Chr8L_17 threads=4 memory=125 k=17 output Chr8L_17_meryllist
../meryl count Chr8L_18 threads=4 memory=125 k=17 output Chr8L_18_meryllist
../meryl count Chr8L_19 threads=4 memory=125 k=17 output Chr8L_19_meryllist
../meryl count Chr8L_20 threads=4 memory=125 k=17 output Chr8L_20_meryllist
../meryl count Chr8L_21 threads=4 memory=125 k=17 output Chr8L_21_meryllist
../meryl count Chr8L_22 threads=4 memory=125 k=17 output Chr8L_22_meryllist
../meryl count Chr8L_23 threads=4 memory=125 k=17 output Chr8L_23_meryllist
../meryl count Chr8L_24 threads=4 memory=125 k=17 output Chr8L_24_meryllist
../meryl count Chr8L_25 threads=4 memory=125 k=17 output Chr8L_25_meryllist
../meryl count Chr8L_26 threads=4 memory=125 k=17 output Chr8L_26_meryllist
../meryl count Chr8L_27 threads=4 memory=125 k=17 output Chr8L_27_meryllist
../meryl count Chr8L_28 threads=4 memory=125 k=17 output Chr8L_28_meryllist
../meryl count Chr8L_29 threads=4 memory=125 k=17 output Chr8L_29_meryllist
../meryl count Chr8L_30 threads=4 memory=125 k=17 output Chr8L_30_meryllist
../meryl count Chr8L_31 threads=4 memory=125 k=17 output Chr8L_31_meryllist
../meryl count Chr8L_32 threads=4 memory=125 k=17 output Chr8L_32_meryllist
../meryl count Chr8L_33 threads=4 memory=125 k=17 output Chr8L_33_meryllist
../meryl count Chr8L_34 threads=4 memory=125 k=17 output Chr8L_34_meryllist
../meryl count Chr8L_35 threads=4 memory=125 k=17 output Chr8L_35_meryllist
../meryl count Chr8L_36 threads=4 memory=125 k=17 output Chr8L_36_meryllist
../meryl count Chr8L_37 threads=4 memory=125 k=17 output Chr8L_37_meryllist
../meryl count Chr8L_38 threads=4 memory=125 k=17 output Chr8L_38_meryllist
../meryl count Chr8L_39 threads=4 memory=125 k=17 output Chr8L_39_meryllist
../meryl count Chr8L_40 threads=4 memory=125 k=17 output Chr8L_40_meryllist
../meryl count Chr8L_41 threads=4 memory=125 k=17 output Chr8L_41_meryllist
../meryl count Chr8L_42 threads=4 memory=125 k=17 output Chr8L_42_meryllist
../meryl count Chr8L_43 threads=4 memory=125 k=17 output Chr8L_43_meryllist
../meryl count Chr8L_44 threads=4 memory=125 k=17 output Chr8L_44_meryllist
../meryl count Chr8L_45 threads=4 memory=125 k=17 output Chr8L_45_meryllist
../meryl count Chr8L_46 threads=4 memory=125 k=17 output Chr8L_46_meryllist
../meryl count Chr8L_47 threads=4 memory=125 k=17 output Chr8L_47_meryllist
../meryl count Chr8L_48 threads=4 memory=125 k=17 output Chr8L_48_meryllist
../meryl count Chr8L_49 threads=4 memory=125 k=17 output Chr8L_49_meryllist
../meryl count Chr8L_50 threads=4 memory=125 k=17 output Chr8L_50_meryllist
../meryl count Chr8L_51 threads=4 memory=125 k=17 output Chr8L_51_meryllist
../meryl count Chr8L_52 threads=4 memory=125 k=17 output Chr8L_52_meryllist
../meryl count Chr8L_53 threads=4 memory=125 k=17 output Chr8L_53_meryllist
../meryl count Chr8L_54 threads=4 memory=125 k=17 output Chr8L_54_meryllist
../meryl count Chr8L_55 threads=4 memory=125 k=17 output Chr8L_55_meryllist
../meryl count Chr8L_56 threads=4 memory=125 k=17 output Chr8L_56_meryllist
../meryl count Chr8L_57 threads=4 memory=125 k=17 output Chr8L_57_meryllist
../meryl count Chr8L_58 threads=4 memory=125 k=17 output Chr8L_58_meryllist
../meryl count Chr8L_59 threads=4 memory=125 k=17 output Chr8L_59_meryllist
../meryl count Chr8L_60 threads=4 memory=125 k=17 output Chr8L_60_meryllist
../meryl count Chr8L_61 threads=4 memory=125 k=17 output Chr8L_61_meryllist
../meryl count Chr8L_62 threads=4 memory=125 k=17 output Chr8L_62_meryllist
../meryl count Chr8L_63 threads=4 memory=125 k=17 output Chr8L_63_meryllist
../meryl count Chr8L_64 threads=4 memory=125 k=17 output Chr8L_64_meryllist
../meryl count Chr8L_65 threads=4 memory=125 k=17 output Chr8L_65_meryllist
../meryl count Chr8L_66 threads=4 memory=125 k=17 output Chr8L_66_meryllist
../meryl count Chr8L_67 threads=4 memory=125 k=17 output Chr8L_67_meryllist
../meryl count Chr8L_68 threads=4 memory=125 k=17 output Chr8L_68_meryllist
../meryl count Chr8L_69 threads=4 memory=125 k=17 output Chr8L_69_meryllist
../meryl count Chr8L_70 threads=4 memory=125 k=17 output Chr8L_70_meryllist
../meryl count Chr8L_71 threads=4 memory=125 k=17 output Chr8L_71_meryllist
../meryl count Chr8L_72 threads=4 memory=125 k=17 output Chr8L_72_meryllist
../meryl count Chr8L_73 threads=4 memory=125 k=17 output Chr8L_73_meryllist
../meryl count Chr8L_74 threads=4 memory=125 k=17 output Chr8L_74_meryllist
../meryl count Chr8L_75 threads=4 memory=125 k=17 output Chr8L_75_meryllist
../meryl count Chr8L_76 threads=4 memory=125 k=17 output Chr8L_76_meryllist
../meryl count Chr8L_77 threads=4 memory=125 k=17 output Chr8L_77_meryllist
../meryl count Chr8L_78 threads=4 memory=125 k=17 output Chr8L_78_meryllist
../meryl count Chr8L_79 threads=4 memory=125 k=17 output Chr8L_79_meryllist
../meryl count Chr8L_80 threads=4 memory=125 k=17 output Chr8L_80_meryllist
../meryl count Chr8L_81 threads=4 memory=125 k=17 output Chr8L_81_meryllist
../meryl count Chr8L_82 threads=4 memory=125 k=17 output Chr8L_82_meryllist
../meryl count Chr8L_83 threads=4 memory=125 k=17 output Chr8L_83_meryllist
../meryl count Chr8L_84 threads=4 memory=125 k=17 output Chr8L_84_meryllist
../meryl count Chr8L_85 threads=4 memory=125 k=17 output Chr8L_85_meryllist
../meryl count Chr8L_86 threads=4 memory=125 k=17 output Chr8L_86_meryllist
../meryl count Chr8L_87 threads=4 memory=125 k=17 output Chr8L_87_meryllist
../meryl count Chr8L_88 threads=4 memory=125 k=17 output Chr8L_88_meryllist
../meryl count Chr8L_89 threads=4 memory=125 k=17 output Chr8L_89_meryllist
../meryl count Chr8L_90 threads=4 memory=125 k=17 output Chr8L_90_meryllist
../meryl count Chr8L_91 threads=4 memory=125 k=17 output Chr8L_91_meryllist
../meryl count Chr8L_92 threads=4 memory=125 k=17 output Chr8L_92_meryllist
../meryl count Chr8L_93 threads=4 memory=125 k=17 output Chr8L_93_meryllist
../meryl count Chr8L_94 threads=4 memory=125 k=17 output Chr8L_94_meryllist
../meryl count Chr8L_95 threads=4 memory=125 k=17 output Chr8L_95_meryllist
../meryl count Chr8L_96 threads=4 memory=125 k=17 output Chr8L_96_meryllist
../meryl count Chr8L_97 threads=4 memory=125 k=17 output Chr8L_97_meryllist
../meryl count Chr8L_98 threads=4 memory=125 k=17 output Chr8L_98_meryllist
../meryl count Chr8L_99 threads=4 memory=125 k=17 output Chr8L_99_meryllist

```
Now get intersection between MOMunique and each of the kmer db from each section of Chr8L
```
./meryl intersect-sum MOMunique bitXXX output MOMunique_bitXXX
```

Now print out the kmers
```
../meryl print testintersect/ > test.out
```

To get the total number of kmers in each Chrbit use a bash loop:
```
for file in *meryllist
do
	printf "$file " >> total_kmerz.out
  ../meryl print "$file" | wc -l >> total_kmerz.out
done
```

To get the total number of kmers in each Chrbit_intersection with Mom_unique use a bash loop:
```
for file in MOMunique*meryllist_intersect
do
	printf "$file " >> total_Mom_uniqueintersection_kmerz.out
  ../meryl print "$file" | wc -l >> total_Mom_uniqueintersection_kmerz.out
done
```

Now print out the kmers (meryl print) and convert to a fasta file like this:
```
cut -f1 MOMunique_Chr8L_20_kmerz.txt > temp.fasta
```
```
perl -pe 's/\n/\n>\n/g' temp.fasta > MOMunique_Chr8L_20_kmerz.fasta
```
then add a greater than sign in the beginning and delete the one at the end and then assemble like this:
```
/home/ben/trinityrnaseq-2.1.1/Trinity --seqType fa --single MOMunique_Chr8L_20_kmerz.fasta --CPU 6 --max_memory 20G --KMER_SIZE 13 --min_contig_length 10 --min_glue 1
```

# Abyss assembly

On graham, load these modules and execute abyss
```
module load abyss/2.0.2
module load gcc/7.3.0
module load openmpi/3.1.2
```
```
abyss-pe name=Mom_chr8L_20 se=MOMunique_Chr8L_20_kmerz.fasta k=16 c=1

abyss-pe k=64 name=Mom_chr8L_20 in='./MOMunique_Chr8L_20_kmerz_extract_both/BJE3897_Mom_LOO1_left_paired_scythe.filtered.fastq ./MOMunique_Chr8L_20_kmerz_extract_both/BJE3897_Mom_LOO1_right_paired_scythe.filtered.fastq' 
```
you can check what the longest contig is (the length is second value in the header; the third value is the converage) easily like this:
```
cut -d" " -f2 Mom_chr8L_20-3.fa | sort -n | tee >(echo "max=$(tail -1)")
```

# Extracting reads based on a kmer list

The abyss assemblies were shitty because theyonly include kmers that have unique sites in the female but not the flanking sequences that are also useful. So I am going to use CookieCutter to pull out raw reads based on kmers and then assemble these reads.  The latest version of Cookiecutter has a bug so I instead had to locally install version 1 release: 'https://github.com/ad3002/Cookiecutter/blob/master/create_package.sh'

This would not install on info so I put it in iqaluk and graham here respectively:
```
/work/ben/2018_Austin_XB_genome
```
and 
```
/home/ben/project/ben/2018_Austin_XB_genome/
```

The first step is to make a kmer library (library.txt) which looks like this:
```
TACCTGAGTAGGCCTAGAAATAAACATGC	1
GTTTATTTCTAGGCCTACTCAGGTAAAAA	1
ATTTTTTACCTGAGTAGGCCTAGAAATAA	1
```
I think the second column of the meryl print output needs to be replaced with a value of 1.

```
awk {$2="1"}1' temp.txt > library.txt	awk '{printf("%s\t%s\n", $2, $1)}’ intersection.txt > intersection.library
```	

and then replace the space with a tab:	

```	
sed -i 's/ /\t/g' library.txt	
```

I did Cookiecutter on iqaluk because I had problems installing it on info. To extract reads on iqaluk, first load python:
```
module load python/core-gcc630/2.7.14
```
and then use this command
```
/work/ben/2018_Austin_XB_genome/Cookiecutter/bin/cookiecutter extract -i BJE3897_Mom_allleft.fastq.gz -f MOMunique_Chr8L_20_kmerz_cookielibrary.txt -o MOMunique_Chr8L_20_kmerz_extract
```
in this directory:
```
/work/ben/2018_Austin_XB_genome/Trimmed_reads
```

I then did the assembly with abyss on graham because it isn't available as a module on iqaluk

And then I made a blast db on info because blast isn't on computecanada:
in this directory: `/2/scratch/ben/2018_Austin_XB_genome/XL_mRNA`:
```
makeblastdb -in xlaevisMRNA.fasta -input_type fasta -dbtype nucl -out xlaevisMRNA_blastable
```
```
blastn -query XXX -db XXX_blastable -outfmt 6 -out XXX -evalue 1e-20 -task megablast
```
```
blastn -query Mom_chr8L_20-contigs.fa -db XL_mRNA/xlaevisMRNA_blastable -outfmt 6 -out XL_mRNA/Mom_chr8L_20-contigs_to_XL_mRNA -evalue 1e-20 -task megablast
```

Get longest alignment length like this:
```
awk  'BEGIN{max=0}{if(($4)>max)  max=($4)}END {print max}' XL_mRNA/Mom_chr8L_20-contigs_to_XL_mRNA
```

# More notes, including on AR

I checked how many kmers are in the MOM and DAD unique_17 mers like this:
```
./meryl print MOM_unique_17 | wc -l
```
for mom, there are: 945,803,526
for dad, there are: 1,088,771,912

So the Dad seems to be more polymorphic (?)

I used these commands to make kmer dbs for the hypervariable region of AR isoforms 1 and 2, and then find the intersection of unique kmers in each isoform with the unique kmers in the MOM or DAD.
```
./meryl count XB_AR_isoform1_hypervar.fasta threads=4 memory=128 k=17 output XB_AR_isoform1_hypervar_meryl
./meryl count XB_AR_isoform2_hypervar.fasta threads=4 memory=128 k=17 output XB_AR_isoform2_hypervar_meryl
./meryl difference XB_AR_isoform1_hypervar_meryl XB_AR_isoform2_hypervar_meryl output iso1_minus_iso2_meryl
./meryl difference XB_AR_isoform2_hypervar_meryl XB_AR_isoform1_hypervar_meryl output iso2_minus_iso1_meryl
./meryl intersect-sum MOM_unique_17 iso1_minus_iso2_meryl output MOM_unique_iso1_minus_iso2_meryl_intersection
./meryl intersect-sum MOM_unique_17 iso2_minus_iso1_meryl output MOM_unique_iso2_minus_iso1_meryl_intersection
./meryl intersect-sum DAD_unique_17 iso1_minus_iso2_meryl output DAD_unique_iso1_minus_iso2_meryl_intersection
./meryl intersect-sum DAD_unique_17 iso2_minus_iso1_meryl output DAD_unique_iso2_minus_iso1_meryl_intersection
./meryl print MOM_unique_iso1_minus_iso2_meryl_intersection > MOM_unique_iso1_minus_iso2_meryl_intersection.fasta
./meryl print MOM_unique_iso2_minus_iso1_meryl_intersection > MOM_unique_iso2_minus_iso1_meryl_intersection.fasta
./meryl print DAD_unique_iso1_minus_iso2_meryl_intersection > DAD_unique_iso1_minus_iso2_meryl_intersection.fasta
./meryl print DAD_unique_iso2_minus_iso1_meryl_intersection > DAD_unique_iso2_minus_iso1_meryl_intersection.fasta

```
Weird result - the Mom_unique_17 had no intersection with unique kmers from either isoform (except one that spanned a splice site and thus is not legit.  But the DAD_unique_17 had many (>50) kmers in common with each isoform. WTF?  I then checked if any of these unique isoform kmers that only matched dad are on chr8L and none are. But the AR does map to chr8L (bit 15).



