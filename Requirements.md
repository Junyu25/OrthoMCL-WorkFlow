## Requirements

(1) UNIX
  - The OrthoMCL Pairs program has only been tested on UNIX, specifically RedHat 5.8.  Other UNIX variants may work, but we can't support them.
  - The MCL program is UNIX compatible only

(2) BLAST
  - we recommend NCBI BLAST for two reasons
     a) theoretically: XXXXXXX
     b) practically: NCBI BLAST supports a tab delimited output which we provide parsers for.  See Step 7 below.
  - for large datasets (e.g. 1M proteins) all-v-all BLAST will likely require a compute cluster. Otherwise, it could run for weeks or months. 

(3) Relational Database 
  - The OrthoMCL Pairs program runs in a relational database.  Supported vendors are:
     - Oracle
     - MySql
  - If you don't already have one of these installed, install MySql, which can be done for free and without significant systems administration support.  (Follow the instructions we provide below.)
  
   We realize that it is a little inconvenient to require a relational database.  However, using a relational database as the core technology for orthomclPairs provides speed, robustness and scalability that would have been very hard to acheive without it.

(4) Hardware
  - the hardware requirements vary dramatically with the size of your dataset.
  - for the Benchmark Dataset, we recommend:
     - memory: at least 4G
     - disk: 100G free space.  
  - you can estimate your disk space needs more accurately when you have completed Step 8 below.  You will need at least 5 times the size of the blast results file produced in that step.  90% of the disk space required is to hold that file, load it into the database and index it in the database.

(5) Perl
  - standard perl
  - DBI libraries

(6) MCL program
  - please see http://www.micans.org/mcl/sec_description1.html

(7) Time
  - The Benchmark Dataset took:
    - 3 days to run all-v-all BLAST on a 500 cpu compute cluster.  
    - 16 hours for the orthmclPairs processing to find pairs
    - 2 hours for MCL to find the groups
