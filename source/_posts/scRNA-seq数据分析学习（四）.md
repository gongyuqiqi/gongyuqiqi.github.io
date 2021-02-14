---
title: scRNA-seq数据分析学习（四）
date: 2020-02-15 16:30:23
tags: NGS
---

学习R包monocle对scRNA-seq数据进行分析

<!--more-->

# scRNA-seq数据分析（四）

## 创建数据集

后续分析的前提就是将数据构建成monocle需要的对象  
因此这里先介绍一下monocle需要的用来构建CellDataSet对象的三个数据集   
- 表达量矩阵`exprs`：数据矩阵，行名是基因，列明是细胞编号
- 细胞的表型信息`phenoData`：第一列是细胞编号，其他列是细胞的相关信息
- 基因注释`featureData`：第一列是基因编号，其他列是基因对应的信息
这三个数据集要满足如下要求  
**表达量矩阵**
- 保证它的列数等于`phenoData`的行数
- 保证它的行数等于`featureData`的行数
- `phenoData`的行名需要和表达矩阵的列明匹配
- `featureData`和表达矩阵的行名要匹配
- `featureData`至少要有一列"gene_short_name"，就是基因的symbol  

```R
rm(list = ls())#clear the environment
options(warn=-1)#turn off warning message globally
suppressMessages(library(monocle))
```

**这里同样使用scRNAseq R包中的数据集，构建monocle对象**

```R
#首先需要一个表达矩阵，还需要临床信息
library(scRNAseq)
data(fluidigm)#load examaple data
assay(fluidigm)<-assays(fluidigm)$rsem_counts#set assay to RSEM estimated counts
ct<-floor(assays(fluidigm)$rsem_counts)
ct[1:4,1:4]#表达矩阵
sample_ann<-as.data.frame(colData(fluidigm))#临床信息
```

```R
#准备monocle对象需要的phenotype和feature data以及表达矩阵，从scRNA-seq这个R包里面提取这三个数据
gene_ann<-data.frame(
  gene_short_name=row.names(ct),
  row.names = row.names(ct)
)
pd<-new("AnnotatedDataFrame",data=sample_ann)
fd<-new("AnnotatedDataFrame",data=gene_ann)
```

#构建monocle后续分析的所有对象，主要是根据包的说明书，仔细探索其需要的构建对象的必备元素
#因为表达矩阵是counts值，所以注意expressionFamily参数,其他类型的值用其他参数

```R
sc_cds<-newCellDataSet(
  ct,
  phenoData = pd,
  featureData = fd,
  expressionFamily = negbinomial.size(),
  lowerDetectionLimit = 1
)
sc_cds
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/sc_sds.JPG"/>

<font color=red>**在此示范如何加载加载RPKM数据，在此并不运行该部分代码**</font>  
从本地读入RPKM值文件，构造CellDataSet对象  

```R
#读入所需要的数据：表达矩阵，样本信息，基因信息
expression_matrix<-read.table("fpkm_matrix.txt")
sample_sheet<-read.delin("cell_sample_sheet.txt")
gene_annotation<-read.delin("gene_annotation.txt")
#创建CellDataSet对象
pd<-new("AnnotatedDataFrame",data=sample_sheet)
fd<-new("AnnotatedDataFrame",data=gene_annotation)
sc_cds<-newCellDataSet(as.matrix(expression_matrix),
                       phenoData=pd,featureData=fd)
```

monocle和scater、seurat他们基于的对象不一样，所以monocle还提供了转换函数  

```R
lymphomadata<-sc_cds
#转换成seurat对象
lymphomadata_seurat<-exportCDS(sc_cds,'Seurat')
#转换成SCESet对象
lymphomadata_scater<-exportCDS(sc_cds,'Scater')
```

**下面是monocle对新构建的CellDataSet对象的标准操作**

```R

library(dplyr)
colnames(phenoData(sc_cds)@data)
#estimateDispersions这步的时间和电脑配置密切相关,所以我们这套分析不选用monocle包自带的数据集，因为太大了，仍然采用scRNAseq内置数据集
sc_cds<-estimateSizeFactors(sc_cds)#一定要运行
sc_cds<-estimateDispersions(sc_cds)#一定要运行
```
# 质控  
对基因和细胞进行质量控制，指控指标根据课题进行具体探索，没有统一标准，这里只是简单演示 

**过滤基因**

```R
cds<-sc_cds
cds
cds<-detectGenes(cds,min_expr = 0.1)#设置基因最小表达量为0.1
print(head(fData(cds)))
expressed_gene<-row.names(subset(fData(cds),num_cells_expressed>=5))
length(expressed_gene)
cds<-cds[expressed_gene,]
cds
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/filtergene.JPG"/>


**过滤细胞**

```R
#过滤细胞
print(head(pData(cds)))#如果没有运行上述步骤的estimateSizeFactors、estimateDispersions，这一步就会发现某个属性出现NA值
tmp<-pData(cds)
fivenum(tmp[,1])
fivenum(tmp[,30])
#这里展示就先不过滤细胞了，有需要就自行过滤
valid_cell<-row.names(pData(cds))#其实这里没有过滤细胞，valid_cell全为TRUE
cds<-cds[,valid_cell]
cds
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/filtercell.JPG"/>


## 聚类  
单细胞转录组最重要的就是把细胞分群，相关的算法非常多，这里选用最常用的tSNE  

```R
plot_pc_variance_explained(cds,return_all = F)
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/plotPC.JPG"/>


```R
#由上图决定用前6个组成份进行降维,降维采用tSNE的方法
cds<-reduceDimension(cds,max_components = 2,num_dim=6,
                     reduction_method = 'tSNE',verbose = T)
cds<-clusterCells(cds,num_clusters = 4)
plot_cell_clusters(cds,1,2,color="Biological_Condition")
table(pData(cds)$Biological_Condition)
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/plottsne.JPG"/>


<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/tableBiologicalananotation.JPG"/>

**可以看出，用monocle的方法进行单细胞转录组数据分析，GW的三群细胞依然无法很好的区分开，NPC倒是聚类得非常完美**  

## 寻找差异基因  

```R
#这一步需要计算量，如果数据量大，这一步很耗时
Sys.time()
diff_test_res<-differentialGeneTest(cds,fullModelFormulaStr = "~Biological_Condition")
Sys.time()
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/difftestres.JPG"/>

**选出差异基因**

```R
sig_gene<-subset(diff_test_res,qval<0.1)#筛选条件qval<0.1
head(sig_gene[,c("gene_short_name","pval","qval")])
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/headdiffgene.JPG"/>

```R
choosed_gene<-as.character(head(sig_gene$gene_short_name))
plot_genes_jitter(cds[choosed_gene,],grouping = "Biological_Condition",ncol = 2)
plot_genes_jitter(cds[choosed_gene,],grouping = "Biological_Condition",
                  color_by = "Biological_Condition",nrow = 3,ncol = NULL
                  )
#自己画图也可以
 #boxplot(log10(exprs(cds)['A1BG',]+1)~pData(cds)$Biological_Condition)
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/plotdiffgenehead.JPG" width="49%" height="50%">
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/plotdiffgeneheadcol.JPG" width="50%" height="50%">  


## 推断发育轨迹 <font color=red>这是monocle包最大的亮点</font>  
**第一步：挑选合适的基因**  
有多种方法，例如可以提供已经的基因集，这里选取统计学上有差异的基因   

```R
ordering_gene<-row.names(subset(diff_test_res,qval< 0.01))
cds<-setOrderingFilter(cds,ordering_gene)
plot_ordering_genes(cds)
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/plotoderinggenes.JPG" width="60%" height="50%">

**第二部：降维**
降维的目的是更好的展示数据。函数里提供很多方法，不同方法最后展示的图会有所不同，DDRTree是Monocle2使用的默认方法

```R
cds<-reduceDimension(cds,max_components = 2,method='DDRTree')
```

**第三步：对细胞进行排序**

```R
cds<-orderCells(cds)
plot_cell_trajectory(cds,color_by = "Biological_Condition")
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/trace.JPG" width="60%" height="50%">


**第四步：可视化marker gene随发育阶段变化的表达情况**

```R
#plot_genes_in_pseudotime可以展示marker基因，本例子随便选取了6个差异表达基因
plot_genes_in_pseudotime(cds[choosed_gene,],color_by = "Biological_Condition")
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/tracemaker.JPG"/>


## 参考资料  
生信技能数（[B站](https://search.bilibili.com/all?keyword=%E7%94%9F%E4%BF%A1%E6%8A%80%E8%83%BD%E6%A0%91&from_source=nav_search&spm_id_from=333.851.b_696e7465726e6174696f6e616c486561646572.9)、公众号）   
