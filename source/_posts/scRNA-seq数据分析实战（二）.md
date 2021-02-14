---
title: scRNA-seq数据分析实战（二）
date: 2020-02-08 16:30:23
tags: NGS
---

体验Seraut包处理GSE111229的scRNA-seq数据

<!--more-->

```R
rm(list = ls()) 
Sys.setenv(R_MAX_NUM_DLLS=999)
```

# 载入数据  

```R
load(file='../input.Rdata')
counts=a
# using raw counts is the easiest way to process data through Seurat.
counts[1:4,1:4];dim(counts)
library(stringr) 
meta=df
head(meta) 
# 下面的基因是文章作者给出的
gs=read.table('top18-genes-in-4-subgroup.txt')[,1]
gs
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/loaddata.JPG"/>


```R
#简单了解一下表达矩阵
fivenum(apply(counts,1,function(x) sum(x>0) ))
boxplot(apply(counts,1,function(x) sum(x>0) ))
fivenum(apply(counts,2,function(x) sum(x>0) ))
hist(apply(counts,2,function(x) sum(x>0) ))
```

## 构建Seraute对象

```R
library(Seurat)
# 其中 min.cells 和 min.genes 两个参数是经验值
#Seurat真的很优秀呀，给定参数，自动进行质控
seu <- CreateSeuratObject(raw.data = counts, 
                          meta.data =meta,
                          min.cells = 5, 
                          min.genes = 2000, 
                          project = "seu")
seu
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/szseuratseu1.JPG"/>


```R
#可以尝试以下手动质控
table(apply(counts,2,function(x) sum(x>0) )>2000)
table(apply(counts,1,function(x) sum(x>0) )>4)
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/manuqc.JPG"/>

可以看到，两种质控方法得到的数据非常接近

**查看对象的基本情况**

```R
head(seu@meta.data) 
dim(seu@data)
```
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/szseuratseu@.JPG"/>

**可视化基本信息**

```R
VlnPlot(object = seu, 
        features.plot = c("nGene", "nUMI" ), 
        group.by = 'plate',
        nCol = 2)
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/szseuratVlnPlotnbyplat.JPG"/>

可以看出：不管是各个样本检测到的基因数量还是文库大小，都没有批次效应。

```R
VlnPlot(object = seu, 
        features.plot = c("nGene", "nUMI" ), 
        group.by = 'g',
        nCol = 2)
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/szseuratVlnPlotbyg.JPG"/>

可以看出：层次聚类得到的分组主要是因为样本间检测的基因数量的差异造成的。

**可以给seu对象增加一个属性，供QC使用**  

```R
#构建新增的属性
ercc.genes <- grep(pattern = "^ERCC-", x = rownames(x = seu@raw.data), value = TRUE)
percent.ercc <- Matrix::colSums(seu@raw.data[ercc.genes, ]) / Matrix::colSums(seu@raw.data)
# 添加新增的属性
seu <- AddMetaData(object = seu, metadata = percent.ercc,
                   col.name = "percent.ercc")
VlnPlot(object = seu, 
        features.plot = c("nGene", "nUMI", "percent.ercc" ), 
        group.by = 'g',
        nCol = 3)
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/szseuratVlnPlotbygeneumiercc.JPG"/>


可以看出：细胞能检测到的基因数量与其含有的ERCC序列反相关。当组间的文库大小差异可忽略时，外源RNA多，计算后的内源RNA的比例自然是会下降的。


**进一步可视化基本信息**

```R
#VlnPlot(seu,group.by = 'plate',c("nGene","Gapdh","Bmp3"))
#上面这段代码和下面的代码时同样的效果
VlnPlot(object = seu, 
        features.plot = c("nGene", "Gapdh", "Bmp3" ), 
        group.by = 'plate',
        nCol = 3)
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/szseuratVlnPlotgenegapdhbmp3.JPG"/>
再次证明没有批次效应

```R
GenePlot(object = seu,  gene1 = "nUMI", gene2 = "nGene")
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/szseuratGenePlotgeneumi.JPG"/>


```R
GenePlot(object = seu,  gene1 = "Brca1", gene2 = "Brca2")
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/szseuratGenePlotbrca12.JPG"/>

```R
CellPlot(seu,seu@cell.names[3], 
         seu@cell.names[4],
         do.ident = FALSE)
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/szseuratCellPlotcell12.JPG"/>


## 表达矩阵标准化

```R
seu <- NormalizeData(object = seu, 
                     normalization.method = "LogNormalize", 
                     scale.factor = 10000)
seu@data[1:4,1:4]
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/szsueratnormalize.JPG"/>


## 挑选变化的基因

```R
# 这里需要理解  dispersion 值
seu <- FindVariableGenes(object = seu, 
                         mean.function = ExpMean, 
                         dispersion.function = LogVMR )
## 默认值是：x.low.cutoff = 0.1, x.high.cutoff = 8, y.cutoff = 1
## 根据经验阈值挑选的变化基因个数。
length( seu@var.genes)
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/szsueratnormalize.JPG"/>


## 去除一些技术误差，比如 nUMI或者ERCC

```R
head(seu@meta.data) 
seu <- ScaleData(object = seu, 
                 vars.to.regress = c("nUMI",'nGene',"percent.ercc" ))
# 后面就不需要考虑ERCC序列了。
seu@scale.data[1:4,1:4]
pheatmap(as.matrix(seu@scale.data[gs,])) 
```

## 普通PCA降维  

用前面挑选的有变化的基因进行降维

```R
seu <- RunPCA(object = seu, pc.genes = seu@var.genes, 
              do.print = TRUE, pcs.print = 1:5, 
              genes.print = 5)
tmp <- seu@dr$pca@gene.loadings
head(tmp)
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/szseuratpc1-20.JPG"/>


```R
VizPCA( seu, pcs.use = 1:2)
```

```R
PCAPlot(seu, dim.1 = 1, dim.2 = 2,group.by = 'plate')
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/szseuratpcaplotbyplate.JPG"/>


```R
PCAPlot(seu, dim.1 = 1, dim.2 = 2,group.by = 'g')
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/szseuratpcaplotbyg.JPG"/>


**根据不同组成份进行热图的展示**

```R
seu <- ProjectPCA(seu, do.print = FALSE)
#根据PC1进行可视化
PCHeatmap(object = seu, pc.use = 1, cells.use = 100, 
          do.balanced = TRUE, label.columns = FALSE)
#使用pc.use = 1:10即批量展示前10各组成份的热图信息
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/szseuratheatmappc1.JPG"/>

**根据参数来调整最后的分组**

```R
#根据参数来调整最后的分组个数
#因为文章分为4群细胞，所以这里调整resolution的值为0.4，最后会得到4个组
#resolution很重要很重要很重要
seu1 <- FindClusters(object = seu, reduction.type = "pca", 
                    dims.use = 1:20, force.recalc = T,
                    resolution = 0.4, print.output = 0, 
                    save.SNN = TRUE)
PrintFindClustersParams(seu1)
table(seu1@meta.data$res.0.4)
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/szseuratfindcluster.JPG"/>


## tSNE展示分组信息  

```R
## resolution 是最关键的参数
seu=seu1
seu <- RunTSNE(object = seu, dims.use = 1:10, do.fast = TRUE)
# note that you can set do.label=T to help label individual clusters
TSNEPlot(object = seu, do.label=T)
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/szseurattSNEplotbypca.JPG" width="60%" height="60%">

## 对每个类别的细胞寻找marker基因

```R
seu.markers <- FindAllMarkers(object = seu, only.pos = TRUE, 
                              min.pct = 0.25, 
                              thresh.use = 0.25)
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/seumaker0.JPG"  width="60%" height="60%">
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/seumaker1.JPG"  width="60%" height="60%">
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/seumaker2.JPG"  width="60%" height="60%">
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/seumaker3.JPG"  width="60%" height="60%">


```R
library(dplyr)
top20 <- seu.markers %>% group_by(cluster) %>% top_n(20, avg_logFC)
#读入作者的4个亚群的top级的差异基因
gs=read.table('top18-genes-in-4-subgroup.txt')[,1]
gs
intersect(top10$gene,gs)
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/intersectgene.JPG"/>

- **这时，你会发现用Seurat方法计算得到的差异基因和作者计算（自己写函数，并没有用到任何R包，有点搞不懂他为什么要这么做，还有一点小小的敬佩）得到的差异基因部分重合（37个共同基因）。**
- **用不同的方法计算得到的差异基因都会有一定的差异，就转录组差异分析而言，limma、DESeq2、EdgeR三大差异分析包计算的差异基因也是部分重合的。**
- **生信分析在一定程度上给研究者提供了方向，具体验证还是要落实到实验上。但是，干数据也很重要，没有方向怎么能行！所以呀，干数据和湿数据都要要，一个都不能少！**

```R
top20 <- seu.markers %>% group_by(cluster) %>% top_n(20, avg_logFC)
intersect(top20$gene,gs)
DoHeatmap(object = seu, genes.use = top20$gene, 
          slim.col.label = TRUE, remove.key = TRUE)
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/DoHeatMap.JPG"/>

```R
FeaturePlot(object = seu, 
            features.plot ="Rgs5", 
            cols.use = c("yellow", "red"), 
            reduction.use = "tsne",
            no.legend=FALSE
            )
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/Rgs5.JPG" width="60%" height="60%">


```R
FeaturePlot(object = seu, 
            features.plot ="Dcn", 
            cols.use = c("yellow", "red"), 
            reduction.use = "tsne",
            no.legend=FALSE
            )
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/Dcn.JPG" width="60%" height="60%">


```R
FeaturePlot(object = seu, 
            features.plot ="Pbk", 
            cols.use = c("yellow", "red"), 
            reduction.use = "tsne",
            no.legend=FALSE
            )
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/Pbk.JPG" width="60%" height="60%">


```R
FeaturePlot(object = seu, 
            features.plot ="Trf", 
            cols.use = c("yellow", "red"), 
            reduction.use = "tsne",
            no.legend=FALSE
            )
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/Trf.JPG" width="60%" height="60%">



