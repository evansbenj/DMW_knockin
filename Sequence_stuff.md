# Organizing sequence data from genome and Yoshimoto et al. 2008

DMW is on scaffold 78 of XL version 9.0.  Here is a command to get this scaffold:

awk -v seq="Scaffold78" -v RS='>' '$1 == seq {print RS $0}' Xla.v91.repeatMasked.fa > scaffold78.fa

