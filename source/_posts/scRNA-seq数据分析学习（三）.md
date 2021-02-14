---
title: scRNA-seq数据分析学习（三）
date: 2020-02-15 16:30:23
tags: NGS
---

学习R包seurat对scRNA-seq数据进行分析

<!--more-->

# scRNA-seq数据分析（三）
😊seurat不是一个R包，seurat是一个优秀的R包  
😊seurat不是提供服务的，seurat是提供一条龙服务的   
- seurat版本有2.×.×和3.×.×。同一般的R包升级不太一样的是：2.×和3.×之间区别还是蛮多的，各种函数也都有变化，虽然升级带来了更多的优点，但是函数名称的变化就会给学习者带来不小的麻烦呀！
- 这里先学习2.×。为什么呢？因为我还不会3.×呀😵😏😜

**载入R包**

```R
rm(list = ls())#clear the environment
options(warn=-1)#turn off warning message globally
library(Seurat)
```

**同样使用scRNAseq内置数据集**  

```R
library(scRNAseq)
data(fluidigm)#加载测试数据
assay(fluidigm)<-assays(fluidigm)$rsem_counts
ct<-floor(assays(fluidigm)$rsem_counts)
ct[1:4,1:4]
counts<-ct
```

## 创建Seurat要求的对象

```R
names(metadata(fluidigm))
meta<-as.data.frame(colData(fluidigm))
identical(rownames(meta),colnames(counts))#检测meta和counts这两个对象，后面有需要
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/identicalmetacounts.JPG"/>


```R
seu<-CreateSeuratObject(raw.data = counts,
                            meta.data = meta,
                            min.cells = 3,
                            min.genes = 200,
                            project = "seu")
seu
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/seu1.JPG"/>


**增加相关属性信息**

```R
#增加线粒体基因信息，如果线粒体所占基因比例过高，意味着可能是死细胞
mito.gene<-grep(pattern = "^MT-",x=rownames(x=seu@data),value = TRUE)#但是我们知道这个数据集里面并没有线粒体基因
percent.mito<-Matrix::colSums(seu@raw.data[mito.gene,])/Matrix::colSums(seu@raw.data)
seu<-AddMetaData(object = seu,metadata = percent.mito,col.name = "percent.mito")#加入了线粒体基因的信息
#这里也可以加入ERCC等其他属性
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/mitoinfo.JPG"/>


## 可视化（初步）

```R
#可视化，meta信息里面存在分组变量，可以指定分组，scRNAseq数据集里的分组信息是"Biological_Condition"
VlnPlot(object = seu,features.plot = c("nGene","nUMI","percent.mito"),group.by = 'Biological_Condition',nCol = 3)
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/VlnPlot.JPG"/>


```R
#查看一下某两个属性间的相关性
GenePlot(object=seu,gene1="nUMI",gene2="nGene")
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/GenePlotnUMInGene.JPG"/>


```R
#查看基因间的相关性
tail(sort(Matrix::rowSums(seu@raw.data)))
GenePlot(object = seu,gene1 = "SOX11",gene2 = "EEF1A1")
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/tailsortgene.JPG"/>

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/GenePlotnUMInGene.JPG"/>


```R
#细胞间的相关性，更好的方式是用cor()+heatmap展示
CellPlot(seu,seu@cell.names[3],seu@cell.names[5],do.identify=FALSE)
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/CellPlot.JPG"/>


## 表达矩阵归一化  
只有进行归一化后，样本之间的比较才更能说明问题 

```R
identical(seu@raw.data,seu@data)
seu<-NormalizeData(object = seu,normalization.method = "LogNormalize",
                   scale.factor = 10000,display.progress = F)
#将每个细胞中总UMI设定为10000，计算方法为loge（每个细胞中基因的nUMI/该细胞内总UMI*10000+1）
#经过归一化后，seu对象里面的data被改变了
identical(seu@raw.data,seu@data)
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/normalizetruefalse.JPG"/>


## 寻找波动比较明显的基因，后续使用这些差异基因进行分析，主要为了降低计算量

```R
seu<-FindVariableGenes(object = seu,mean.function = ExpMean,dispersion.function = LogVMR,
                       x.low.cutoff = 0.0125,
                       y.high.cutoff = 3,
                       y.cutoff = 0.5)
#选择不同的阈值，得到的基因取决于实际情况
length(seu@var.genes)
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/fvg.JPG"/>

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/lengthfvg.JPG"/>


## 对归一化后的矩阵进行分群
- 对矩阵进行回归建模，以及scale
- center=T：每个细胞中基因表达量-该基因在所有细胞中的表达量
- scale=T：每个细胞中基因中心化后的表达值/该基因所在所有细胞中表达值的标准差
- 注意：执行ScaleData之前需要先执行NormalizeData

```R
seu<-ScaleData(object = seu,vars.to.regress = c("nUMI"),display.progress = F)#在这里只去除了文库大小的影响
```

## PCA降维   

```R
#采用上述的差异基因进行降维
seu<-RunPCA(object = seu,
            pc.genes = seu@var.genes,
            do.print = 1:5,
            genes.print = 5)
seu@dr
#这样就能拿到PC的基因的重要性占比情况
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/pcareduction.JPG"/>


```R
tmp<-seu@dr$pca@gene.loadings
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/pcainfo.JPG"/>


```R
#看一下前两个主成分的情况
VizPCA(seu,pcs.use = 1:2)
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/pca1pca2.JPG"/>


```R
PCAPlot(seu,dim.1=1,dim.2=2,group.by='Biological_Condition')
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/pca1pca2BC.JPG"/>


```R
#热图查看PC1的情况
PCHeatmap(object = seu,pc.use = 1,cells.use = ncol(seu@data),do.balanced = TRUE,label.columns = FALSE)
#一次性展示前10个主成分在各样本间的体现情况
#PCHeatmap(object = seu,pc.use = 1:10,cells.use = ncol(seu@data),do.balanced = TRUE,label.columns = FALSE)

```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/PCA1heatmap.JPG"/>


基于PCA情况看看细胞如何分群  
重点：需要搞清楚**resolution**参数

```R
seu<-FindClusters(object = seu,
                  reduction.type = "pca",
                  dims.use = 1:10,force.recalc = T,
                  resolution = 0.9,print.output = 0,
                  save.SNN = TRUE)
PrintFindClustersParams(seu)
table(seu@meta.data$res.0.9)
```

**resolution**的值调得不一样，最后table出来得细胞亚群数也会不一样

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/pcaresulution.JPG"/>


## 细胞分群后的tSNE图  

```R
seu<-RunTSNE(object = seu,
             dims.use = 1:10,
             do.fast=TRUE,
             perplexity=10)
TSNEPlot(object = seu)#由图看出，tSNE分出了3群，为什么呢？因为上面PCA降维出3各亚群呀！
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/tsneafterpca.JPG"/>

**同样是PCA降维算法得到得3各细胞亚群，tSNE明显展示出更好的试图效果**  

```R
table(meta$Biological_Condition)
table(meta$Biological_Condition,seu@meta.data$res.0.9)
#这里就有一个问题了：这里明明在探讨tSNR降维，为什么这里又用到了上面pca降维中所生成的参数res.0.9呢？
#有一个办法，就是先不进行pca降维，tSNE降维看看是否依然得到当前的数据
#问题来了，我验证了上一条猜想，结果呢：Error in GetDimReduction(object = object, reduction.type = reduction.use,  : 
                                        #pca  dimensional reduction has not been computed
#所以呢，在这里的tSNE降维居然要用到PCA降维，嗯......表示不是很理解呀！两者不是独立的吗！？！？
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/grouppcaBC.JPG"/>


## 对每个亚群寻找marker基因  

下面的代码需要适时修改，因为每次分组都不一样，本次是3组，因为pca降维成3组  
**以第一群细胞为例`ident.1 = 1`**  

```R
markers_df<-FindMarkers(object = seu,ident.1 = 1,min.pct = 0.25)
print(x=head(markers_df))
markers_genes=rownames(head(x=markers_df,n=5))
vlnPlot(object=seu,features.plot=markers_genes,use.raw=TRUE,y.log=TRUE)
FeaturePlot(object = seu,
            features.plot = markers_genes,
            cols.use = c("grey","blue"),
            reduction.use = "tsne")
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/headmarker_df.JPG"/>

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/VlnPlotcluster1marker.JPG"/>


<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/FeaturePlotcluster1marker.JPG"/>


## 展示各个分类的marker基因的表达情况

```R
seu.markers<-FindAllMarkers(object = seu,only.pos = TRUE,min.pct = 0.25,thresh.use=0.25)
DT::datatable(seu.markers)#这个操作有点优秀呀，可以在Rstudio里面试一试
```

## 热图展示各个亚群的marker基因

```R
library(dplyr)
seu.markers%>%group_by(cluster)%>%top_n(2,avg_logFC)
#每个亚群挑10个marker基因进行展示
top10<-seu.markers%>%group_by(cluster)%>%top_n(10,avg_logFC)
DoHeatmap(object = seu,genes.use = top10$gene,slim.col.label = TRUE,remove.key = TRUE)
#FeaturePlot批量产图
#FeaturePlot(object = seu,features.plot = top10$gene,cols.use = c("grey","blue"),reduction.use = "tsne")
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/clustersheatmap.JPG"/>


