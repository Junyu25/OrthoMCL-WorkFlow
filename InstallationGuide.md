## Steps in detail

### Step 1: Install and configure the relational database 安装配置Mysql
[Mysql安装](http://gitlab.bioinfo.site/Junyu/orthomcl/blob/master/MysqlInstallGuide.md)   
[Mysql配置](http://gitlab.bioinfo.site/Junyu/orthomcl/blob/master/MysqlConfigurationGuide.md)


### Step 2: install mcl
Get the latest software from http://www.micans.org/mcl/src/mcl-latest.tar.gz

Follow the install instructions.

Also see: http://www.micans.org/mcl/sec_description1.html


### Step 3: install and configure OrthoMCL programs 安装配置 OrthoMCL
Input:
  - orthomclSoftware.tar
Output:
  - directory of executable programs
  - home directory for your run of orthomcl
  - orthomcl.config file

Use this command to unpack the software:
  tar -xf orthomclSoftware.tar

The result will be this:
  orthomclSoftware/
    bin/
     ...
    doc/
     UserGuide.txt
     orthomcl.config.template
    lib/

The bin/ directory has a set of programs.  To run the programs you will need to either:
  a) include the orthomclSoftware/bin directory in your PATH
  b) call the programs using their full directory path

Make a directory to hold the data and results for your run of orthomcl.  In this document we will call that directory "my_orthomcl_dir".

In the orthomclSoftware/doc/Main/OrthoMCLEngine directory is a file called orthomcl.config.template.  Copy that file to my_orthomcl_dir/orthomcl.config, and edit the new file according to the following instructions.

In the examples below, it is assumed that your MySql server has a database called 'orthomcl'.  You can either create one (go into the server and run 'create database orthomcl') or use an existing database, and change the dbConnectString accordingly.

dbVendor=
  - either 'oracle' or 'mysql'
  - used by orthomclInstallSchema, orthomclLoadBlast, orthomclPairs
dbConnectString=
  - the string required by Perl DBI to find the database.  
  - examples are:
     dbi:Oracle:orthomcl                 (for an oracle database with service name 'orthomcl')
     dbi:MySql:orthomcl                  (for a centrally installed mysql server with a database called 'orthomcl')
     dbi:MySql:orthomcl:localhost:3307   (for a user installed mysql server on port 3307 with a database called 'orthomcl')
  - used by orthomclInstallSchema, orthomclLoadBlast, orthomclPairs, orthomclDumpPairsFiles
dbLogin=
  - your database login name
  - used by orthomclInstallSchema, orthomclLoadBlast, orthomclPairs, orthomclDumpPairsFiles
dbPassword=
  - your database password
  - used by orthomclInstallSchema, orthomclLoadBlast, orthomclPairs, orthomclDumpPairsFiles
similarSequencesTable=
  - the name to give the table that will be loaded with blast results by orthomclLoadBlast. This is configurable for your flexibility.  It doesn't matter what you call it. 
  - used by orthomclInstallSchema, orthomclLoadBlast, orthomclPairs
orthologTable=
  - the name of the table that will hold potential ortholog pairs.  This is configurable so that you can run orthomclPairs multiple times, and compare results.
  - used by orthomclInstallSchema, orthomclPairs, orthomclDumpPairsFiles
inParalogTable=InParalog
  - the name of the table that will hold potential inparalog pairs.  This is configurable so that you can run orthomclPairs multiple times, and compare results.
  - used by orthomclInstallSchema, orthomclPairs, orthomclDumpPairsFiles
coOrthologTable=CoOrtholog
  - the name of the table that will hold potential coortholog pairs.  This is configurable so that you can run orthomclPairs multiple times, and compare results.
  - used by orthomclInstallSchema, orthomclPairs, orthomclDumpPairsFiles
interTaxonMatchView=InterTaxonMatch
percentMatchCutoff=
  - blast similarities with percent match less than this value are ignored.
  - used by orthomclPairs
evalueExponentCutoff=
  - blast similarities with evalue Exponents greather than this value are ignored.
  - used by orthomclPairs
oracleIndexTblSpc=
  - optional table space to house all oracle indexes, if required by your oracle server.  default is blank.


### Step 4: orthomclInstallSchema 数据库架构
Input:
  - database   
  
Output:
  - database with schema installed

Run the orthmclInstallSchema program to install the schema. (Run the program with no arguments to get help.  This is true of all following orthomcl programs.)
```shell
orthomclSoftware-v2.0.9$ bin/orthomclInstallSchema my_orthomcl_dir/orthomcl.config.template my_orthomcl_dir/install_schema.log
```

Benchmark time: < 1 min
