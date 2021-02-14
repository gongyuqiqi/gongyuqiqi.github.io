---
title: scRNA-seq数据分析实战（三）
date: 2020-02-22 16:30:23
tags: NGS
---

single cell RNA-seq  && 10×   
利用Seurate 2.3.3完成单细胞转录组下游分析，重现文章《Acquired cancer resistance to combination immunotherapy from transcriptional loss of class I HLA》的部分图片。

<!--more-->

```R
rm(list = ls()) # clear the environment
#load all the necessary libraries
options(warn=-1) # turn off warning message globally
suppressMessages(library(Seurat))
```

## 读入文章关于第一个病人的PBMC表达矩阵

```R
raw_dataPBMC <- read.csv('path/to/GSE117988_raw.expMatrix_PBMC.csv.gz', header = TRUE, row.names = 1)
raw_dataPBMC[1:4,1:4]
dim(raw_dataPBMC)
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/dataPBMC.JPG"/>


## 标准化

```R
dataPBMC <- log2(1 + sweep(raw_dataPBMC, 2, median(colSums(raw_dataPBMC))/colSums(raw_dataPBMC), '*')) # Normalization，矩阵大也会比较耗时
```

## 按照治疗阶段分组（后续会用到）

```R
timePoints <- sapply(colnames(dataPBMC), function(x) ExtractField(x, 2, '[.]'))

timePoints <-ifelse(timePoints == '1', 'PBMC_Pre', 
                    ifelse(timePoints == '2', 'PBMC_EarlyD27',
                           ifelse(timePoints == '3', 'PBMC_RespD376', 'PBMC_ARD614')))
table(timePoints)
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/tabletimepoint.JPG"/>

## 表达矩阵的基本信息可视化（可用于知道质控）

```R
## 表达矩阵的质量控制
fivenum(apply(dataPBMC,1,function(x) sum(x>0) ))
boxplot(apply(dataPBMC,1,function(x) sum(x>0) ))
fivenum(apply(dataPBMC,2,function(x) sum(x>0) ))
hist(apply(dataPBMC,2,function(x) sum(x>0) ))
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/fibenumdataPBMC.JPG"/>

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/boxplotdataPBMC.JPG" width="70%" height="70%">

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/histdataPBMC.JPG" width="70%" height="70%">

## 然后创建Seurat的对象


```R
# Create Seurat object
# already normalized (above)
PBMC <- CreateSeuratObject(raw.data = dataPBMC, 
                           min.cells = 1, min.genes = 0, project = '10x_PBMC') #没有进行过滤
PBMC 
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/PBMC.JPG"/>

## 根据需要添加metadata信息

```R
# Add meta.data (nUMI and timePoints)
PBMC <- AddMetaData(object = PBMC, 
                    metadata = apply(raw_dataPBMC, 2, sum),
                    col.name = 'nUMI_raw')
PBMC <- AddMetaData(object = PBMC, metadata = timePoints, col.name = 'TimePoints')
```

## 将对象进行基本的可视化

```R
sce=PBMC
VlnPlot(object = sce, 
        features.plot = c("nGene", "nUMI"), 
        group.by = 'TimePoints', nCol = 2)
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/VlnPlotsce.JPG"/>


```R
GenePlot(object = sce, gene1 = "nUMI", gene2 = "nGene")
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/GenePlotsce.JPG"/>

```R
#可以看看高表达量基因是哪些
tail(sort(Matrix::rowSums(sce@raw.data)))
## 散点图可视化任意两个基因的一些属性（通常是细胞的度量）
# 这里选取两个基因。
tmp=names(sort(Matrix::rowSums(sce@raw.data),decreasing = T))
GenePlot(object = sce, gene1 = tmp[1], gene2 = tmp[2])
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/GenePlotgene.JPG"/>

```R
# 散点图可视化任意两个细胞的一些属性（通常是基因的度量）
# 这里选取两个细胞
CellPlot(sce,sce@cell.names[3],sce@cell.names[4],do.ident = FALSE)
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/GenePlotsample.JPG"/>


## 聚类可视化（重点）

```R
# Cluster PBMC
PBMC <- ScaleData(object = PBMC, vars.to.regress = c('nUMI_raw'), model.use = 'linear', use.umi = FALSE)
#因为矩阵比较大，scale这一步非常耗内存，计算机内存不够的话这一步会报错
PBMC <- FindVariableGenes(object = PBMC, mean.function = ExpMean, dispersion.function = LogVMR, x.low.cutoff = 0.0125, x.high.cutoff = 3, y.cutoff = 0.5)
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/fvg.JPG"/>


```R
PBMC <- RunPCA(object = PBMC, pc.genes = PBMC@var.genes)
##

## 避免太多log日志被打印出来。
PBMC <- FindClusters(object = PBMC, 
                     reduction.type = "pca", 
                     dims.use = 1:10, 
                     resolution = 1, 
                     print.output = 0,
                     k.param = 35, save.SNN = TRUE)
#在此，resolution、k.param的值设定不同，分群效果也会不一样，参数的调整也是一个比较麻烦的事情
table(PBMC@ident)# 分出了13个clusters
PBMC <- RunTSNE(object = PBMC, dims.use = 1:10)
TSNEPlot(PBMC, colors.use = c('green4', 'pink', '#FF7F00', 'orchid', '#99c9fb', 'dodgerblue2', 'grey30', 'yellow', 'grey60', 'grey', 'red', '#FB9A99', 'black'),do.label=T)
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/tSNE.JPG" width="70%" height="70%">


```R
table(PBMC@meta.data$TimePoints,PBMC@ident)
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/timepointident.JPG"/>


## 添加亚群信息

```R
current.cluster.ids <- c(0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12)
#具体是哪一种cell type,根据每个亚群的maker基因，再结合生物学背景确定亚群的具体类型，这个也有网站预测。
new.cluster.ids <- c("B cells", "CD14+ T cells", "Classical monicytes","Naive memory T cells", "CD8 effector T cells", "NK cells", "NULL1", "Non-classical monocytes","Dendritic cells", "NULL2","CD8+ cytotoxic T cells","Myeloid cells","NULL3")
PBMC@ident <- plyr::mapvalues(x = PBMC@ident, from = current.cluster.ids, to = new.cluster.ids)
TSNEPlot(object = PBMC, colors.use = c('green4', 'pink', '#FF7F00', 'orchid', '#99c9fb', 'dodgerblue2', 'grey30', 'yellow', 'grey60', 'grey', 'red', '#FB9A99', 'black'),do.label=T)
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/tsnelabelcellname.JPG"/>



## <font color=red>QUESTION</font>
- 代码都跟原文的代码一样，虽然主要想体现的东西是有所体现的，聚类出来的细胞亚群还是有小小的偏差。
- `FindClusters`的参数真的是我至今为止最最纠结的问题啦，参数不一样，亚群就会不一样，难不成为了得到自己想要的分群效果，不停的调整参数吗？
- 对象问题也是个巨大的问题，对象里面包裹了好多东西，有些不太会调用。
- 该数据按照TimePoint来体现，目前自创的方法无法做到与原文类似的图，这也是一个待解决的问题。