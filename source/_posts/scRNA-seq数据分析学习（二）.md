---
title: scRNA-seq数据分析学习（二）
date: 2020-02-14 16:30:23
tags: NGS
---

学习R包scater对scRNA-seq数据进行分析

<!--more-->

# scRNA-seq数据分析（二）

```R
rm(list = ls())#clear the environment
options(warn=-1)#turn off warning message globally
```

# 认识scater
## 创建scater要求的对象

```R
library(scRNAseq)
data(fluidigm)
assay(fluidigm)<-assays(fluidigm)$rsem_counts
ct<-floor(assays(fluidigm)$rsem_counts)
ct[1:4,1:4]
table(rownames(ct)==0)
sample_ann<-as.data.frame(colData(fluidigm))#数据集的临床信息
#这里需要将表达矩阵做成我们的scater要求的对象
sce<-SingleCellExperiment(
  assays=list(counts=ct),
  colData=sample_ann
)
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/scepre.JPG"/>


## 过滤
**基因层面的过滤**

使用`calculateQCMetrics`函数作用于sce这个单细胞数据对象后，就可以用`rowData(object)`查看各个基因各项统计指标：  
- `mean_counts`：平均表达量
- `log10_mean_counts`：归一化 log10-scale
- `pct_dropout_by_counts`：该基因丢失率
- `n_cell_by_counts`：多少个细胞表达了该基因  

上面的指标可以用来过滤，也可以自己计算这些统计学指标  
主要是过滤掉**低表达量的基因**，还有**线粒体基因**和**ERCC spike-ins**的控制

```R
exprs(sce)<-log2(calculateCPM(sce)+1)
genes<-rownames(rowData(sce))#rowData(object)基因相关统计情况
genes[grepl('^MT-',genes)]
genes[grepl('^ERCC-',genes)]
#比较不幸，这个测试数据里面没有线粒体基因，也没有ERCC序列
sce<-calculateQCMetrics(sce,
                        feature_controls = list(ERCC=grep('^ERCC',genes)))
#后面的分析都是基于sce这个变量，这个是一个SingleCellExperiment对象
#sce是一个list(S4),但这个list可以当成data.frame来使用，非常方便
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/MTERCC.JPG"/>



```R
#查看信息
tmp<-as.data.frame(rowData(sce))
colnames(tmp)
head(tmp)
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/sceproperty.JPG"/>


```R
#目前只过滤掉那些在所有细胞都没有表达的基因
#这个过滤条件可以自行调整，可以看到基因数量大幅减少
keep_feature<-rowSums(exprs(sce)>0)>0
table(keep_feature)
sce<-sce[keep_feature,]
sce
``` 

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/scemid.JPG"/>


**细胞层面的过滤**  
用`colData(object)`可以查看各个样本统计情况
```R
tmp<-as.data.frame(colData(sce)) 
colnames(tmp)
#尝试查看一些信息
tf<-sce$total_features_by_counts
boxplot(tf)
fivenum(tf)
#还是那句话，过滤没有统一的标准，视具体情况而定，下面举一个例子进行简单的过滤尝试
tmp$pct_counts_in_top_100_features_endogenous<50
table(tmp$pct_counts_in_top_100_features_endogenous<50)
sce<-sce[,tmp$pct_counts_in_top_100_features_endogenous<50]
sce
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/filtercountsboxplot.JPG" width="30%" height="30%">

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/scefin.JPG"/>

**这个sce真的是非常方便了，虽然是个list,但是却可以像dataframe一样对其进行处理，**
**至此，我们过滤掉了9000多个基因，4个样本**

## 数据可视化

```R
#展示一些基因在不同细胞分类的表达，这里只做一下演示，一般不这么用，不过筛出来的marker基因可以用这种方式展现其在各组间的差异
plotExpression(sce,rownames(sce)[1:6],
               x="Biological_Condition",
               exprs_values = "logcounts")
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/6genesin4groups.JPG" width="70%" height="70%">


**PCA**
```R
#还可以可视化细胞距离分布
sce<-runPCA(sce)#这里没有进行任何基因的挑选，就直接进行PCA了，与seurat包不一样
reducedDimNames(sce)
#PCA分布图上添加临床信息，同样发现不同GW细胞之间不能很好的分群
#plotPCA(sce)该数据最原始的PCA图
plotReducedDim(sce,use_dimred = "PCA",colour_by = "Biological_Condition")
```
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/scePCA.JPG" width="70%" height="70%">

```R
#PCA分布图上面添加表达量信息，一般标marker gene，次方法不常用，这里就以一个基因为例示范一下
plotReducedDim(sce,use_dimred = "PCA",colour_by = rownames(sce)[1],size_by = rownames(sce)[1])
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/scePCAcertaingene.JPG" width="60%" height="60%">


```R
##仅仅是选取前20个PC，分群时细胞分布并没有太大区别
sce<-runPCA(sce,ncomponents = 20)
plotPCA(sce,colour_by="Biological_Condition")
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/scePCAcomponent20.JPG" width="70%" height="70%">

```R
##可以挑选指定的PC来可视化，这里选5个PC
plotPCA(sce,ncomponents=5,colour_by="Biological_Condition")
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/PCA5component.JPG"/>

**tSNE**

```R
#tSNE可视化
set.seed(1234)
sce<-runTSNE(sce,perplexity = 30)
plotTSNE(sce,colour_by="Biological_Condition")
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/tsne.JPG" width="70%" height="70%">

```R
#perplexity的设定值不一样，散点的分布也会有所不同，但趋势都是一致的
#perplexity设置为10
sce<-runTSNE(sce,perplexity = 10,use_dimred = "PCA",n_dimred = 10)
plotTSNE(sce,colour_by="Biological_Condition")
#perplexity设置为20
sce<-runTSNE(sce,perplexity = 20,use_dimred = "PCA",n_dimred = 10)
plotTSNE(sce,colour_by="Biological_Condition")
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/scePCAper10.JPG" width="50%" height="50%"><img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/scePCAper20.JPG" width="50%" height="50%">

# 认识SC3
## R包SC3处理scRNAseq内置的数据

```R
#R包SC3
library(SC3)
sce<-sce_back
sce<-sc3_estimate_k(sce)
metadata(sce)$sc3$k_estimation
rowData(sce)$feature_symbol<-rownames(rowData(sce))
#一步运行sc3的所有分析，相当耗时，数据量大的话就真的相当耗时
#这里kn表示的预估聚类数，数据集是已知的，我们这里就设定为4组，具体数据要具体考虑
kn<-4
sc3_cluster<-"sc3_4_clusters"
sce<-sc3(sce,ks=kn,biology = TRUE)
```

## 可视化展示
kn就是聚类数

```R
#热图：比较先验分类和SC3的聚类的一致性
sc3_plot_consensus(sce,k=kn,show_pdata = c("Biological_Condition",sc3_cluster))
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/sc3pheatmapconsensus.JPG"/>


```R
#热图：展示表达量信息
sc3_plot_expression(sce,k=kn,show_pdata = c("Biological_Condition",sc3_cluster))
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/sc3pheatmapexpression.JPG"/>

<font color=red>**SC3包找marker基因**</font>

```R
#热图，展示可能的标记基因
sc3_plot_markers(sce,k=kn,show_pdata =c("Biological_Condition",sc3_cluster) )
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/sc3pheatmarker.JPG"/>



