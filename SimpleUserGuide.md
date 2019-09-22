# 简明中文教程
## 1~4 [Installation](http://gitlab.bioinfo.site/Junyu/orthomcl/blob/master/InstallationGuide.md) 略
已安装，未添加环境变量，请复制到自己所需路径下使用。
```shell
cp -R /home/cjy/Downloads/orthomclSoftware-v2.0.9 .
```

## 5. orthomclAdjustFasta

```shell
#处理 fasta 格式文件
$/home/scr/02_software/orthomclSoftware-v2.0.9/bin/orthomclAdjustFasta pdel out_02/pdel.pep.fa 1  
# argv[1] 表示修改后每条序列的开头名称及文件名, argv[2]表示取原始序列名称的第一部分作为名称, 两个名称之间用'|'连接
# 将要做同源分析的物种逐个处理
```
## 6. orthomclFilterFasta

```shell
#序列筛选(The filter is based on length and percent stop codons)
$/home/scr/02_software/orthomclSoftware-v2.0.9/bin/orthomclFilterFasta out_02 10 20
# argv[1] 为存放上一步结果的目录, argv[2] 表示蛋白序列最小允许长度, 官方建议为10, argv[3] 表示终止密码子最大比例, 官方建议为20
```
## 7. All-v-all BLAST
```shell
#blastp, 最好时间, 可以拆分一下再比对
$makeblastdb -in out_03/goodProteins.fasta -dbtype prot -out out_04/orthomcl
$blastp -query out_03/goodProteins.fa -out out_04/orthomcl_blastp.out -db out_04/orthomcl -evalue 1e-5 -num_threads 5
```
## 8. orthomclBlastParser
```shell
#处理比对结果, 用于提交给orthomcl database
$/home/scr/02_software/orthomclSoftware-v2.0.9/bin/orthomclBlastParser out_04/orthomcl_blastp.out out_02 >> out05/similarSequences.txt
#要根据结果文件大小修改my.cnf中参数, 太累了,不想写了, 参考mysqlConfigurationGuide.txt改吧
```
## 9. orthomclLoadBlast
```shell
#提交给orthomcl database
$/home/scr/02_software/orthomclSoftware-v2.0.9/bin/orthomclLoadBlast out_01/00.orthomcl.config out_05/ilarSequences.txt
```
## 10. orthomclPairs
```shell
#这一步是主要的计算环节, 用于找到配对的蛋白
$/home/scr/02_software/orthomclSoftware-v2.0.9/bin/orthomclPairs out_01/00.orthomcl.config out_07/orthomcl_pairs.log cleanup=no
#第二次运行时往往会出错, 要把之前的database删掉, 重跑上边的命令
```

## 11. orthomclDumpPairsFiles
```shell
#获得ortholog, coortholog, inparalog文件
$/home/scr/02_software/orthomclSoftware-v2.0.9/bin/orthomclDumpPairsFiles out_01/00.orthomcl.config
```
## 12. mcl
```shell
#马尔科夫模型聚类算法软件
$mcl mclInput --abc -I 1.5 -o out_09/mclOutput
```

## 13. orthomclMclToGroups
```shell
#输出聚类之后的结果文件
$/home/scr/02_software/orthomclSoftware-v2.0.9/bin/orthomclMclToGroups all_ 1 < out_09/mclOutput > out10/all.txt
```

[参考原文](https://blog.csdn.net/weixin_33978016/article/details/87496305)