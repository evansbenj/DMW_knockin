# Organizing sequence data from genome and Yoshimoto et al. 2008

DMW is on scaffold 78 of XL version 9.0.  Here is a command to get this scaffold:

awk -v seq="Scaffold78" -v RS='>' '$1 == seq {print RS $0}' Xla.v91.repeatMasked.fa > scaffold78.fa

There are about 779,919 base pairs in this scaffold.

# 5' region
The 5 prime upstream region is 3,200 bp long and matches positions 273,956 to 277,110.

# DMW mRNA

The DMW mRNA is 796 bp long, including a polyA tail of 17 bp.

Positions 1 to 27 match 277,111 to 277,137. This is immediately after  the 5' region and must be exon 1.

Positions 27 to 267 of the DMW mRNA matches positions 290,55 to 290,795. This must be exon 2.

Positions 267 to 417 match 295,550 to 295,709.  This much be exon 3.

Positions 423 to 770 of the DMW mRNA matches positions 303,744 to 304,091.  This much be exon 4, including the stop codon.  

Combined, the 5' and mRNA are almost 4000 bp (3996bp).
