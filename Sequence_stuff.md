# Organizing sequence data from genome and Yoshimoto et al. 2008

DMW is on scaffold 78 of XL version 9.0.  Here is a command to get this scaffold:

awk -v seq="Scaffold78" -v RS='>' '$1 == seq {print RS $0}' Xla.v91.repeatMasked.fa > scaffold78.fa

There are about 779,919 base pairs in this scaffold.

The 5 prime upstream region matches positions 273,956 to 277,110.

The end positions 423 to 770 of the DMW mRNA matches positions 303,744 to 304,091.  So this much be the last exon, including the stop codon.  Yoshimoto et al. 2008 fig 2 says DMW has 4 exons; maybe the first three were not sequenced in the genome project? These could be Ns in Scaffold78, but this is unlikely because scaffold78 has only 1537 Ns.
