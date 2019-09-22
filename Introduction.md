## Introduction

For alternate documentation online, please read Unit 6.12 of the Current Protocols of Bioinformatics available at:
  http://onlinelibrary.wiley.com/doi/10.1002/0471250953.bi0612s35/full
   
For details on the orthomcl algorithm, please read the OrthoMCL Algorithm Document: 
  https://docs.google.com/document/d/1RB-SqCjBmcpNq-YbOYdFxotHGuU7RK_wqxqDAMjyP_w/pub

The input to OrthoMCL is a set of proteomes.

The output is a set of files:
>  pairs/
     potentialOrthologs.txt
     potentialCoorthologs.txt
     potentialInparalogs.txt
  groups.txt

The files in the pairs/ directory contain pairwise relationships between proteins, and their scores.  They are categorized into potential orthologs, co-orthologs and inparalogs as described in the OrthoMCL Algorithm Document (see link above).  The groups.txt file contains the groups created by clustering the pairs with the MCL program.  

There are three overall stages:
>  - all-v-all BLAST
>  - the OrthoMCL Pairs program -- makes the pairs/ directory
>  - the MCL program -- clusters the pairs to make the groups.txt file

These stages are executed in a series of thirteen steps detailed below.  Most simply involve running a provided program.  They are broken into steps for ease of backtracking and recoverability.  Most are *very simple* to run, so don't be discouraged.
