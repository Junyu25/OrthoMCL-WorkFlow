# Overview of steps

This is an overview of the thirteen steps to run orthomcl.  Details are in the next sections. 

All programs except mysql and mcl are provided as part of the OrthoMCL download.  The provided programs all begin with 'orthomcl' and will print help if called with no arguments.

>(1) install or get access to a supported relational database.  If using MySql, certain configurations are required, so it may involve working with your MySql administrator or installing your own MySql.  See the mysqlInstallationGuide.txt document provided with the orthomcl software.    

(1) 安装或访问受支持的关系数据库


>(2) download and install the mcl program according to provided instructions.   

(2) 根据提供的说明下载并安装mcl程序


>(3) install and configure the OrthoMCL suite of programs.   

(3) 安装和配置OrthoMCL程序套件


>(4) run orthomclInstallSchema to install the required schema into the database.   

(4) 运行orthomclInstallSchema将所需的架构安装到数据库中。


>(5) run orthomclAdjustFasta (or your own simple script) to generate protein fasta files in the required format.   

(5) 运行orthomclAdjustFasta（或您自己的简单脚本）以所需格式生成蛋白质fasta文件。

>(6) run orthomclFilterFasta to filter away poor quality proteins, and optionally remove alternative proteins. Creates a single large goodProteins.fasta file (and a poorProteins.fasta file)   

(6) 运行orthomclFilterFasta过滤掉劣质蛋白质，并任选去除替代蛋白质。创建一个大的goodProteins.fasta文件（和一个poorProteins.fasta文件）

>(7) run all-v-all NCBI BLAST on goodProteins.fasta (output format is tab delimited text).   

(7) 在goodProteins.fasta上运行all-v-all NCBI BLAST（输出格式是制表符分隔文本）

>(8) run orthomclBlastParser on the NCBI BLAST tab output to create a file of similarities in the required format   

(8) 在NCBI BLAST选项卡输出上运行 orthomclBlastParser，以创建所需格式的相似文件

>(9) run orthomclLoadBlast to load the output of orthomclBlastParser into the database.   

(9) 运行orthomclLoadBlast将 orthomclBlastParser 的输出加载到数据库中

>(10) run the orthomclPairs program to compute pairwise relationships.   

(10) 运行 orthomclPairs 程序来计算成对关系

>(11) run the orthomclDumpPairsFiles program to dump the pairs/ directory from the database   

(11) 运行 orthomclDumpPairsFiles 程序从数据库中转储 pairs /目录

>(12) run the mcl program on the mcl_input.txt file created in Step 11.   

(12) 在步骤11中创建的 mcl_input.txt 文件上运行mcl程序。

>(13) run orthomclMclToGroups to convert mcl output to groups.txt   

(13)运行 orthomclMclToGroups 将mcl输出转换为 groups.txt

We recommend you save the output of each step so that you can easily redo it if things go wrong.