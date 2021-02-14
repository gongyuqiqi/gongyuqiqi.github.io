---
title: scRNA-seq数据分析学习（一）
date: 2020-02-13 16:30:23
tags: NGS
---

用常规转录组的知识对scRNA-seq数据进行初步探索

<!--more-->

# scRNA-seq数据分析学习（一）

<table><tr><td bgcolor=GreenYellow>用常规转录组的知识对scRNA-seq数据进行初步探索</td></tr></table>
  
**bulk RNA-seq和scRNA-seq的区别**  
http://www.bio-info-trainee.com/2548.html  
- 样本量，基因数量导致统计学环境的变化  
  通常情况下样本量都是成千上万的，检测的基因数量相对常规转录组较少
- 计算量迫使算法需要优化  
  成千上万甚至几十万的样本量对计算要求非常高  
- 待解决的生物问题有所不同  
  1. 不仅仅是不同状态的不同，一个样本就是一个因素
  2. 细胞、组织间的异质性问题
  3. 探究变化的过程追踪和重现
- 相对常规转录组而言，scRNA-seq分析流程软件工具的成熟度有待提高，目前还没有对整个分析流程的金标准   

**单细胞差异基因表达统计学方法**  
- 归一化
- 聚类分群
- 重要基因挑选
  1. 差异基因
  2. marker基因
  3. 变异基因

## 安装并加载scRNAseq这个R包  
R包**scRNAseq**内含数据集，下载安装加载相应的辅助R包来探索**scRNAseq**的内置数据集，对单细胞转录组分析进行初步探索（在此之前已经下载好了所需的R包）  
**下载R包**

```R
#如果没有所需R包，可以用以下方法进行下载(用两个R包为例)
options()$repos #查看使用install.packages安装的默认镜像
options("repos"=c(CRAN="https://mirrors.tuna.tsinghua.edu.cn/CRAN/"))#指定install.packages安装镜像为清华镜像
options()$repos
options()$BioC_mirror#查看使用bioconductor的默认镜像
options(BioC_mirror="https://mirrors.ustc.edu.cn/bioc/")#指定Bioconductor安装默认镜像为中科大镜像
options()$BioC_mirror
(!requireNamespace("Rtsne"))#查看是否存在Rtsne这个包
install.packages("Rtsne")#如果Rtsne包不存在，就下载这个R包
#CRAN是R默认使用的R包仓库，install.packages()只能用于安装发布在CRAN上的包
#Bioconductor是基因组数据分析相关的软件仓库包
(!requireNamespace("BiocManager"))
install.packages("BiocManager")#install.packages()是安装Bioconductor软件包的命令
(!requireNamespace("scRNAseq"))
BiocManager::install("scRNAseq")
```



**加载R包**
```R
#suppressMessages(library(Rtsne)) suppressMessages()函数是加载R包的时候不显示说明信息
library(Rtsne)
library(FactoMineR)
library(factoextra)
library(scater)
library(scRNAseq)
library(M3Drop)
library(ROCR)
library(tidyr)
library(cowplot)
library(ggplot2)
```

## scRNA-seq包中的数据集  
这个内置的是Pollen et al.2014年数据集，人类单细胞数据，分为4类：NPC、GW16、GW21、GW21+3。此R包只提供了4种细胞类型，完整的数据是23730 features，301 samples。  
这里面的表达矩阵是有RSEM(Li and Dewey 2011)软件根据hg38 RefSeq transcription得到的，总共130个文库。每个细胞测了2次。测序深度不一样。  

**载入R包scRNA-seq**

```R
library(scRNAseq)
data(fluidigm)
ct<-floor(assays(fluidigm)$rsem_counts)#assays函数拿到表达矩阵，rsem_counts有小数，所以floor
ct[1:4,1:4]
dim(ct) #[1] 26255   130
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/datapartial.JPG"/>


**接下来的代码初步探索scRNA-seq内置数据集的各个属性**

```R
sample_ann<-as.data.frame(colData(fluidigm))#得到130个样本的28个临床信息
#利用lapply针对sample_ann中的前19个属性（数值型的）进行批量处理，一次性展示
box<-lapply(colnames(sample_ann[,1:19]), function(i){
  dat<-sample_ann[,i,drop=F]
  dat$sample=rownames(dat)
  ggplot(dat,aes('all cells',get(i))) +geom_boxplot() +xlab(NULL) +ylab(i) +theme_classic()
})
plot_grid(plotlist = box,ncol = 5)
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/ggplot19.JPG"/>


```R
##基因表达情况的初步探索
counts<-ct
table((apply(counts, 1, function(x) sum(x>0))))#查看基因的详细表达情况
table((apply(counts, 1, function(x) sum(x>0)>0)))#查看至少有一个样本表达某个基因的总体情况
boxplot(apply(counts, 1, function(x) sum(x>0)))
hist(apply(counts,2,function(x) sum(x>0)))#可视化每个样本的基因表达情况
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/countsapply1.JPG"/>

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/countsapplybox1.JPG" width="40%" height="40%">


<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/geneexpression.JPG" width="60%" height="60%">

**过滤（基因+细胞）**

```R
#过滤不满足某个条件的基因
#choosed_genes<-apply(counts, 1, function(x) sum(x>0))>0#等价于choosed_genes<-apply(counts, 1, function(x) sum(x>0)>0)
#table(choosed_genes)
#结果显示
#choosed_genes
#FALSE  TRUE 
#9159 17096
#counts<-counts[choosed_genes,]#过滤掉了在所有样本中都不表达的基因  
#同理也可以过滤掉不满足某个条件的细胞，apply函数中的1改为2，在设定特定的条件即可  
#在此，我们根据上述样本的boxplot图中展示的各个属性来进行简单过滤
#过滤没有统一的标准，根据自己对数据的要求，细节会很多，据不同情况而定
#这里进行简单的批量处理，不考虑太多
filter<-colnames(sample_ann[,c(1:9,11:16,18,19)])
tf<-lapply(filter,function(i){
  #i=filter[1]
  dat<-sample_ann[,i]
  dat<-abs(log10(dat))
  fivenum(dat)
  (up<-mean(dat)+2*sd(dat))
  (down<-mean(dat)-2*sd(dat))
  valid<-ifelse(dat>down & dat<up,1,0)
})
tf<-do.call(cbind,tf)
choosed_cell<-apply(tf, 1, function(x) all(x==1))
table(sample_ann$Biological_Condition)
sample_ann=sample_ann[choosed_cell,]#rownames(sample_ann)中的样本和rownames(tf)数字一一对应
table(sample_ann$Biological_Condition)
ct<-ct[,choosed_cell]#colnames(ct)和rownames(tf)的数字一一对应
dim(ct)#26255个基因，99个样本
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/filter.JPG"/>


## 探索基因的表达情况

```R
ct[1:4,1:4]
counts<-ct
table(apply(counts, 1, function(x) sum(x>0)))
fivenum(apply(counts, 1, function(x) sum(x>0)))
boxplot(apply(counts, 1, function(x) sum(x>0)))
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/matrixexpression.JPG"/>

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/boxplotgeneexpression.JPG" width="40%" height="40%">

```R
#针对基因
fivenum(apply(counts, 2, function(x) sum(x>0)))
hist(apply(counts, 2, function(x) sum(x>0)))
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/fivenumgene.JPG"/>

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/filtergeneexpression.JPG" width="60%" height="60%">


R语言中fivenum函数介绍：https://blog.csdn.net/mr_muli/article/details/79616124  
R语言中do.call函数介绍：https://blog.csdn.net/zdx1996/article/details/87899029  
R语言all函数介绍：https://blog.csdn.net/scong123/article/details/70184038

## 利用常规转录组分析知识查看细胞间所有基因表达量的相关性  

```R 
##下面的计算都是基于log后的表达矩阵
dat<-log2(edgeR::cpm(counts)+1)
dat[1:4,1:4]
dat_back<-dat#备份表达矩阵
exprSet<-dat_back
colnames(exprSet)
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/dat1_4.JPG"/>

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/colnamesexprSet.JPG"/>

**相关性可视化**

```R
pheatmap::pheatmap(cor(exprSet))#查看矩阵中个样本间的相关性
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/pheatmap.JPG"/>


```R
#加上了分组信息注释,组内样本相似性应该高于组间样本相似性
group_list<-sample_ann$Biological_Condition
tmp<-data.frame(group=group_list)
rownames(tmp)<-rownames(sample_ann)
pheatmap::pheatmap(cor(exprSet),annotation_col = tmp)#加上了分组信息注释
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/annotationpheatmap.JPG"/>


```R
#去除掉小于5个样本表达的基因
dim(exprSet)
exprSet=exprSet[apply(exprSet, 1, function(x) sum(x>1)>5),]
dim(exprSet)
pheatmap::pheatmap(cor(exprSet),annotation_col = tmp)
```
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/filter5sampleplusgene.JPG"/>

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/filter5sampleplusgenepheatmap.JPG"/>


```R
#选取前500个差异最大的基因
exprSet<-exprSet[names(sort(apply(exprSet, 1, mad),decreasing = T)[1:500]),]
dim(exprSet)
exprSet[1:4,1:4]
M<-cor(log2(exprSet+1))
tmp<-data.frame(group=group_list)
rownames(tmp)<-colnames(M)
pheatmap::pheatmap(M,annotation_col = tmp)
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/500pheatmap.JPG"/>


- **有热图分析可知，从细胞的相关性角度来看，NPC跟另外的GW细胞群可以区分得很好，但是GW本身得3个小群并没有那么好得区分度**  
- **简单的选取top的sd的基因来计算相关性，并没有明显的改善**  

## 首先对表达矩阵进行简单的层次聚类  

```R
hc<-hclust(dist(t(dat))) #次dat是上述经过过滤的，剩下99个细胞样本的数据
plot(hc,labels = FALSE)
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/hc.JPG"/>

```R
clus<-cutree(hc,4)#对hclust()函数的聚类结果进行剪枝，即选择输出指定类别的系谱聚类结果
group_list<-as.factor(clus)#转换为因子
table(group_list)#统计频数
table(group_list,sample_ann$Biological_Condition)
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/hclustandbiocondition.JPG"/>

**可以看到如果是普通的层次聚类的话，GW16、GW21、GW21+3是很难区分开的**  

## 看看常规PCA的结果  

```R
dat<-dat_back
dat<-t(dat)
dat<-as.data.frame(dat)
anno<-sample_ann$Biological_Condition
dat<-cbind(dat,anno)
dat[1:4,26254:26256]
table(dat$anno)
dat.pca<-PCA(dat[,-ncol(dat)],graph = FALSE)#PCA分析之前先去掉变量即新增加的分组信息anno
head(dat.pca$var$coord)#每个组成份的基因重要性占比
head(dat.pca$ind$coord)#每个细胞的前5个主成分取值
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/prePCA.JPG"/>


```R
fviz_pca_ind(
  dat.pca,
  geom.ind = "point",
  col.ind = dat$anno,
  #color by groups
  addEllipses = TRUE,
  legend.title="groups"
)
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/PCA.JPG"/>
**同样的，NPC和其他类型细胞区分得很好，但GW本身得3个小群体并没有很好得区分度**

## tSNE降维结果

```R
##tSNE降维
#dat_matrix<-dat_back 原始数据会比较耗时，结果就不展示了
dat_matrix<-dat.pca$ind$coord
library(Rtsne)
set.seed(42)
tsne_out<-Rtsne(dat_matrix,perplexity = 10,check_duplicates = FALSE)
plate<-sample_ann$Biological_Condition
plot(tsne_out$Y,col=rainbow(4)[as.numeric(as.factor(plate))])
```


参考资料  
**生信菜鸟团**：http://www.bio-info-trainee.com/  
**生信技能树**：http://www.biotrainee.com/  
**R包的下载方法**：https://www.jianshu.com/p/8e0dece51757