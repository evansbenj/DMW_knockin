# RNAseq

During the spring of 2017, I did an XB cross and dissected out the kidney/mesophrys/gonad complex from about 30 tadpoles at stage 49-51. This is stage that DMW is expressed in XL.  My reasoning is that the sex determining gene of XB might also be expressed at this stage.  I used SOX3 and SF1 primers from BenF's study to genotype male and female tads and then I pooled the tissue from each sex and did RNA extractons using the Qiagen RNeasy extraction kit. 

When the RNAseq data are available (hopefully timorrow), I plan to do the following: 
* Do a joint transcriptome assembly
* Do individual transcriptome assemblies for the male pool and the female pool
* Map reads from each sex to the joint transcriptome assembly
* Identify transcripts that are female (and male) specific
* Blast the sex-specific assemblies against each other and identify transcripts with no hits
* Quantify expression levels using joint transcriptome assembly and HTseq

Hopefully these exercises will help me identify some female-specific candidates. Next step would be to try to amplify the candidate(s) in male and female XB with the hope that I can get a sex-specific amplification. The primers used for this could also be used for checking the CRISPR/Cas mutations.
