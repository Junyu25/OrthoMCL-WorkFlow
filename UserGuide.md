
### Step 1~4: [Installation](http://gitlab.bioinfo.site/Junyu/orthomcl/blob/master/InstallationGuide.md)
已安装，未添加环境变量，请复制到自己所需路径下使用。
```shell
cp -R /home/cjy/Downloads/orthomclSoftware-v2.0.9 .
```
登录mysql：
`mysql -u root -p`
```
drop database orthomcl;
CREATE DATABASE orthomcl;
GRANT SELECT,INSERT,UPDATE,DELETE,CREATE VIEW,CREATE, INDEX, DROP on orthomcl.* TO orthomcl@localhost;
```

```shell
../bin/orthomclInstallSchema ../orthomcl.config ../install_schema.log
```


### Step 5: orthomclAdjustFasta
用 orthomclAdjustFasta 来生成适配的fasta文件
Input: 
>  - fasta files as acquired from the genome resource.

Output:
>  - the my_orthomcl_dir/compliantFasta/ directory of orthomcl-compliant fasta files (see Step 6)

Use orthomclAdjustFasta to produce a compliant file from any input file that conforms to the following pattern (for other files, provide your own script to produce complaint fasta files):
  - has one or more fields that are separated by white space or the '|' character (optionally surrounded by white space)
  - has the unique ID in the same field of every protein.

First, for any organism that has multiple protein fasta files, combine them all into one single proteome fasta file
首先将所有蛋白质的fasta文件整合成单个fasta文件。

Then, create an empty my_orthomcl_dir/compliantFasta/ directory, and change to that directory.  Run orthomclAdjustFasta once for each input proteome fasta file.  It will produce a compliant file in the new directory.  Check each file to ensure that the proteins all have proper IDs.
然后创建my_orthomcl_dir/compliantFasta/目录，转换到这个目录下运行orthomclAdjustFasta.

Benchmark time: < 1 min per genome


实例：
according to orthomcl guide switch to

```shell
orthomclSoftware-v2.0.9$ cd my_orthomcl_dir/compliantFasta/
```
and after cow.fasta enter image description here
```bash
orthomclSoftware-v2.0.9/my_orthomcl_dir/compliantFasta$ ../../bin/orthomclAdjustFasta cow /home/....../cow.fasta 1
orthomclSoftware-v2.0.9/my_orthomcl_dir/compliantFasta$ ../../bin/orthomclAdjustFasta hsa /home/....../human.fasta 1
orthomclSoftware-v2.0.9/my_orthomcl_dir/compliantFasta$ ../../bin/orthomclAdjustFasta mus /home/....../mouse.fasta 1
```
These commandes will create 3 files in compliantFasta directory name cow.fasta, hsa.fasta ans mus.fasta

---

### Step 6: orthomclFilterFasta
>
Input: 
>  - my_orthomcl_dir/compliantFasta/ 
>  - optionally a gene->protein mapping file

Output:
>  - my_orthomcl_dir/goodProteins.fasta
>  - my_orthomcl_dir/poorProteins.fasta
>  - report of suspicious proteomes (> 10% poor proteins)

这一步会生成一个goodProteins.fasta用于BLAST.
This step produces a single goodProteins.fasta file to run BLAST on.  It filters away poor-quality sequences (placing them in poorProteins.fasta).  The filter is based on length and percent stop codons.  You can adjust these values.

The input requirements are:
  1) a compliantFasta/ directory which contains all and only the proteome .fasta files, one file per proteome.
  2) each .fasta file must have a name in the form 'xxxx.fasta' where xxxx is a three or four letter unique taxon code.  For example: hsa.fasta or eco.fasta
  3) each protein in those files must have a definition line in the following format:
     >xxxx|yyyyyyyy
     where xxxx is the three or four letter taxon code and yyyyyyy is a sequence identifier unique within that taxon.

Change dir to my_orthomcl_dir/ and run orthomclFilterFasta.

Benchmark time:  5 min

实例：
```
orthomclSoftware-v2.0.9/my_orthomcl_dir/compliantFasta$ ../../bin/orthomclFilterFasta . 10 20
```
---

### Step 7: All-v-all BLAST
Input:
>  - goodProteins.fasta

Output:
>  - your_blast_results_in_tab_format

You must run your own BLAST.  For large datasets you should consider gaining access to a compute cluster.

We expect you to:
  - use NCBI BLAST
  - run with the -m 8 option to provide tab delimited output required by Step 8
  - for IMPORTANT DETAILS about other BLAST arguments, see:
    the OrthoMCL Algorithm Document (https://docs.google.com/document/d/1RB-SqCjBmcpNq-YbOYdFxotHGuU7RK_wqxqDAMjyP_w/pub)

If you are a power user you can deviate from this, so long as you can ultimately provide output in exactly the format provided by NCBI BLAST using the -m 8 option, and expected by Step 8.

If you are a super-power user you can deviate from that, and also skip Step 8.   But you must be able to provide the exact format file created by that step as expected by Step 9.  The tricky part is computing percent match.  

Time estimate: highly dependent on your data and hardware

实例：
You need first to create a local database in file for blast

```bash
orthomclSoftware-v2.0.9/my_orthomcl_dir/compliantFasta$  makeblastdb -in goodProteins.fasta -dbtype prot -out my_prot_blast_db
```
and

```
orthomclSoftware-v2.0.9/my_orthomcl_dir/compliantFasta$ blastall -p blastp -F 'm S' -v 100000 -b 100000 -z 55 -e 1e-5 -d my_prot_blast_db -i goodProteins.fasta -o out.tab -m 8
```

---


### Step 8: orthomclBlastParser

Input:
>  - your_blast_results_in_tab_format
>  - my_orthomcl_dir/compliantFasta/

Output:
>  - my_orthomcl_dir/similarSequences.txt

这一步将BLAST的结果转换成可加载到orthomcl数据库的文件.
This step parses NCBI BLAST -m 8 output into the format that can be loaded into the orthomcl database. 

Use the orthomclBlastParser program for this.   In addition to formatting, it computes the percent match of each hit, which is tricky (see the perl code if you are a super-power user.)

```shell
orthomclBlastParser my_blast_results compliantFasta >> similarSequences.txt
```
IMPORTANT NOTE: the size of this file determines the disk space required by the relational database.  You will need 5x the size of this file.  Please see the oracleConfigGuide or mysqlConfigGuide now that you know the size of this file.

Benchmark time: 10 min

---

In my own case, I have create a directory name 'blast' compliantFasta directory in copy cow.fasta, hsa.fasta ans mus.fasta in this diectory by doing

```shell
orthomclSoftware-v2.0.9/my_orthomcl_dir/compliantFasta$ mkdir blast
```
and
```shell
orthomclSoftware-v2.0.9/my_orthomcl_dir/compliantFasta$ cp cow.fasta blast
orthomclSoftware-v2.0.9/my_orthomcl_dir/compliantFasta$ cp hsa.fasta blast
orthomclSoftware-v2.0.9/my_orthomcl_dir/compliantFasta$ cp mus.fasta blast
```
and
```shell
orthomclSoftware-v2.0.9/my_orthomcl_dir/compliantFasta$ ../../bin/orthomclBlastParser out.tab blast/ >> similarSequences.txt
```

---


### Step 9: orthomclLoadBlast

Input:
>  - similarSequences.txt

Output:
>  - SimilarSequences table in the database

这一步才将BLAST的结果加载到orthomcl的数据库中
This step loads the BLAST results into the orthomcl database. 

Use the orthomclLoadBlast program for this. 

NOTE: You might get the following error when you run this command:

  "The used command is not allowed with this MySQL version."

The SQL that causes this is LOAD DATA LOCAL INFILE.  MySql needs specific configuration to enable this command.  See these two pages:

http://dev.mysql.com/doc/refman/5.1/en/load-data-local.html
http://dev.mysql.com/doc/refman/5.0/en/loading-tables.html

In sum:
  1) you need to start the MySql server with the option:  --local_infile=1
  2) during installation, MySql needs to be have been compiled with:  --enable-local-infile.  

It is possible that your MySql was compiled with that flag.  It is included by default in some distributions of MySql.   The only way we know of to find out what compile flags were used is to try the mysqlbug command.  This command opens an email so you can report a bug.  Apparently at the bottom of the mail is a list of compile flags.  Once you see them you can abort the mail.  If your mysql was not compiled with that flag it will need to be reinstalled.


Benchmark time: 4 hours

---
```shell
orthomclSoftware-v2.0.9/my_orthomcl_dir/compliantFasta$ ../../bin/orthomclLoadBlast ../orthomcl.config similarSequences.txt
```

---



### Step 10: orthomclPairs
Input:
>  - SimilarSequences table in the database

Output:
>  - PotentialOrthologs table
>  - PotentialInParalogs table
>  - PotentialCoOrthologs table

This is a computationally major step that finds protein pairs.  It executes the algorithm described in the OrthoMCL Algorithm Document (docs.google.com/Doc?id=dd996jxg_1gsqsp6), using a relational database.  The program proceeds through a series of internal steps, each creating an intermediate database table or index.  There are about 20 such tables created. Finally, it populates the output tables.

The cleanup= option allows you to control the cleaning up of the intermediary tables.  The 'yes' option drops the intermediary tables once they are no longer needed.  The 'no' option keeps the intermediary tables in the database.  In total, they are expected to be about 50 percent of the SimilarSequences table. They are useful mostly for power users or developers who would like to query them. They can be removed afterwards with the 'only' or 'all' options.  The latter also removes the final tables, and should only be done after Step 11 below has dumped them to files.

The startAfter= option allows you to pick up where the program left off, if it stops for any reason.  Look in the log to find the last completed step, and use its tag as the value for startAfter=

Because this program will run for many hours, we recommend you run it using the UNIX 'screen' program, so that it does not abort in the middle.  (If it does, use startAfter=).

Benchmark time: 16 hours

---
```shell
orthomclSoftware-v2.0.9/my_orthomcl_dir/compliantFasta$ ../../bin/orthomclPairs ../orthomcl.config ../orthomcl_pairs.log cleanup=no
```

---


### Step 11: orthomclDumpPairsFiles
Input:
>  - database with populated pairs tables

Output
>  - pairs/ directory.  
>  - mclInput file

Run the orthomclDumpPairsFiles.

The pairs/ directory contains three files: ortholog.txt, coortholog.txt, inparalog.txt.  Each of these has three columns:
   - protein A
   - protein B
   - their normalized score (See the Orthomcl Algorithm Document).

Benchmark time: 5 min

---
```shell
orthomclSoftware-v2.0.9/my_orthomcl_dir/compliantFasta$ ../../bin/orthomclDumpPairsFiles ../orthomcl.config
```


### Step 12: mcl
Input:
>  - mclInput file

Output:
>  - mclOutput file

mcl my_orthomcl_dir/mclInput --abc -I 1.5 -o my_orthomcl_dir/mclOutput

Benchmark time: 3 hours

---
```shell
orthomclSoftware-v2.0.9/my_orthomcl_dir/compliantFasta$ mcl mclInput --abc -I 1.5 -o mclOutput
```
---

### Step 13: orthomclMclToGroups
Input:
>  - mclOutput file    

Output:
>  - groups.txt

Change to my_orthomcl_dir and run:
```shell
  orthomclMclToGroups my_prefix 1000 < mclOutput > groups.txt
```

my_prefix is an arbitrary string to use as a prefix for your group IDs.

1000 is an arbitrary starting point for your group IDs.

Benchmark time: 1 min

---
```shell
orthomclSoftware/my_orthomcl_dir/compliantFasta$ ../../bin/orthomclMclToGroups cluster 1000 < mclOutput > groups.txt
```


实例来源：https://www.biostars.org/p/199083/