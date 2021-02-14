---
title: 2020 ATAC-seq analysis
date: 2020-12-08 13:53:55
tags: NGS
---

<!--more-->

# **2020 ATAC-seq analysis**

## 一、ATAC-seq个性化的conda environment的搭建

### 1、git clone

git clone文章中提到的atac-seq-pipeline (里面有你接下来分析所需要的绝大部分东西。另外一定要去看README.md，这个文档会告诉你分析ATAC-seq需要做些准备，进行些什么工作。)

```bash
$ cd ~/ATAC
$ git clone https://github.com/ENCODE-DCC/atac-seq-pipeline
```

### 2、环境搭建

（1）conda base environment搭建
```bash
#一路yes下去
$ wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
$ bash Miniconda3-4.6.14-Linux-x86_64.sh
#最后一个yes时你同意了每次自动启动conda的base环境，显示安装完成后，一定要退出当前的终端，再重新打开一次，你就会发现在游标的最左边多了一个（base）,你的base conda environment就激活啦！你的conda环境搭建就成功了一小步(*^__^*) 嘻嘻……
```

（2）失活掉base conda environment自动激活的设置
```bash
$ conda config --set auto_activate_base false
#同样的完成上诉操作后，记得退出当前终端，再次打开一个新的终端。你就会发现游标的最左边没有（base）了
```

（3）安装pipeline's conda environment

！？你是不是充满了疑问！？下面的两个shell脚本哪里冒出来的，干什么用的。不要忘了，我们第一步git clone了这个atac-seq-pipeline呢！这两个脚本就是就是来源于这个clone下来的pipeline的。

```bash
# 网不给力的朋友，比如说我，这一部就是整个分析流程的限速步骤！！！网络给力这一步也需要花一些时间，读一下两个shell文件就知道为什么啦。
$ bash scripts/uninstall_conda_env.sh  # uninstall it for clean-install
#下面这个shell脚本会创建两个分析ATAC-seq数据所需要的环境，一个基于python3，一个基于python2的
$ bash scripts/install_conda_env.sh
```

（4）激活安装的conda环境

```bash
$ conda activate encode-atac-seq-pipeline
#你一定会疑问为什么是encode-atac-seq-pipeline,阅读一下scripts/install_conda_env.sh这个脚本就知道啦。因为源文档作者把这个环境名命名为encode-atac-seq-pipeline。
```
（5）查看所有的conda环境
```bash
conda env list
```
我们会看到有一个基于python2的环境，没有标明python版本的那个encode-atac-seq-pipeline是基于python3的。有些软件是要基于特定版本的python运行的。
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/conda environment.JPG"/>


## 二、数据下载

```bash
#一次性下载所有的......_1.fastq.gz和......_2.fastq.gz样本
dir=/home/gongyuqi/.aspera/connect/etc/asperaweb_id_dsa.openssh
x=_1
y=_2
for id in {93,98,99}
do
ascp -QT -l 300m -P33001 -i $dir era-fasp@fasp.sra.ebi.ac.uk:/vol1/fastq/SRR126/0$id/SRR126920$id/SRR126920$id$x.fastq.gz .
ascp -QT -l 300m -P33001 -i $dir era-fasp@fasp.sra.ebi.ac.uk:/vol1/fastq/SRR126/0$id/SRR126920$id/SRR126920$id$y.fastq.gz . 
done
```


## 三、bowtie2比对

### 1、 fastqc+multiqc查看原始数据的质量情况
```bash
conda insatll -y fastqc multiqc
cd raw 
fastqc -t 8 *gz -o ./
multiqc *.gzip -o ./
```
multiqc结果表明原始数据质量已经很好了，完全没有adaptor残留,连PCR重复都是勉强过关的。per base sequence content和kmer content两项指标不合格，但是并不会对后续的比对造成多大的负面影响，同时这两项指标通过过滤软件的处理可能并不会后得到改善。这样的数据其实可以直接比对的。
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/multiqc.JPG"/>

### 2、 过滤低质量的reads 
（如果原始数据需要进行过滤处理，那么走下面这个过滤流程，我们这里就不过滤了）

```bash
nohup cat rawdata.txt|while read id
do
arr=(${id})
fq1=${arr[0]}
fq2=${arr[1]}
trim_galore -j 35 --phred33 --length 35 -e 0.1 --stringency 4 --paired -o ../clean $fq1 $fq2
done &
#过滤后的数据质量控制，看数据是否有得到改善，有没有去掉adaptor非常重要
cd clean
fastqc -t 20 *gz -o ./
multiqc *.gzip -o ./
```

### 3、 比对参考基因组
- 比对
```bash
nohup cat sample.txt | while read id
do
arr=(${id})
fq1=${arr[0]}
fq2=${arr[1]}
sample=${fq1%%_*}.sort.bam
bowtie2  --very-sensitive -X2000 --mm --threads 50 -x /home/gongyuqi/ref/mm10/index/mm10 -1 $fq1 -2 $fq2|samtools sort -O bam -@ 15 - -o ../align/$sample 
done &
```
- 比对情况统计及查看
```bash
ls *.bam | while read id
do
samtools flagstat -@ 4 $id  > ${id%%.*}.stat
done
ls *.stat | while read id; do cat $id | grep "mapped (";done
```
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/mapped.JPG"/>

- 生成bam文件的索引文件
```bash
nohup ls *sort.bam|while read id; do samtools index -@ 10 $id;done &
```


## 四、PCR去除重及质控制

### 1、初步过滤 
#=============================  
#Remove unmapped, mate unmapped  
#not primary alignment, reads failing platform  
#Only keep properly paired reads  
#Obtain name sorted BAM file  
#=============================================

```bash
#fixmate要求输入的bam文件按名称排序，所以第一步初步过滤后要按reads名称排序一下
nohup ls *.sort.bam|while read id; do samtools view -@ 8 -F 524 -f 2 -u $id | samtools sort -@ 8 -n /dev/stdin -o ../filter/${id%%.*}.filt.nmsrt.bam;done &
#multimapping=4是bowtie2的默认设置
cd ../filter
multimapping=4 
nohup ls *filt.nmsrt.bam|while read id;do samtools view -@ 8 -h $id|assign_multimappers.py -k $multimapping --paired-end |samtools fixmate -@ 8 -r /dev/stdin ${id%%.*}.filt.nmsrt.fixmate.bam; done &
```
### 2、进一步
#===============================================  
#Remove orphan reads (pair was removed)  
#and read pairs mapping to different chromosomes  
#Obtain position sorted BAM  
#===============================================

```bash
#多个样本写循环
nohup ls *.filt.nmsrt.fixmate.bam|while read id; do samtools view -@ 8 -F 1804 -f 2 -u $id | samtools sort -@ 8 /dev/stdin -o ${id%%.*}.prefilt.bam;done &
```

### 3、标记PCR重复

#===============  
#Mark duplicates  
#===============

```bash
MARKDUP=/home/gongyuqi/miniconda3/envs/encode-atac-seq-pipeline/share/picard-2.20.7-0/picard.jar
#多个样本写循环
nohup ls *.prefilt.bam|while read id;do java -Xmx16G -jar $MARKDUP MarkDuplicates INPUT=$id OUTPUT=${id%%.*}.prefilt.dupmark.bam METRICS_FILE=${id%%.*}.prefilt.dupmark.qc VALIDATION_STRINGENCY=LENIENT ASSUME_SORTED=true REMOVE_DUPLICATES=false;done &
```
### 4、去除PCR重复、建立索引文件、按name排序的bam文件、质控文件

#============================  
#Remove duplicates  
#Index final position sorted BAM  
#Create final name sorted BAM  
#============================

```bash
#多个样本写循环
#去重
nohup ls *.prefilt.dupmark.bam|while read id;do samtools view -@ 16 -F 1804 -f 2 -b $id > ${id%%.*}.nodup.bam;done &
#构建索引文件
nohup ls *nodup.bam|while read id;do samtools index $id;done &
```

### 5、测序文库复杂度的检验
#=============================  
#Compute library complexity  
#=============================  
#Sort by name  
#convert to bedPE and obtain fragment coordinates  
#sort by position and strand  
#Obtain unique count statistics  
#================================================


```bash
#多个样本写循环
nohup ls *.nodup.bam|while read id;do samtools sort -@ 16 -n $id -o ${id%%.*}.nodup.srt.name.bam;done &
nohup ls *.nodup.srt.name.bam|while read id; do bedtools bamtobed -bedpe -i $id | awk 'BEGIN{OFS="\t"}{print $1,$2,$4,$6,$9,$10}' | grep -v 'chrM' | sort | uniq -c | awk 'BEGIN{mt=0;m0=0;m1=0;m2=0} ($1==1){m1=m1+1} ($1==2){m2=m2+1} {m0=m0+1} {mt=mt+$1} END{m1_m2=-1.0; if(m2>0) m1_m2=m1/m2;printf "%d\t%d\t%d\t%d\t%f\t%f\t%f\n",mt,m0,m1,m2,m0/mt,m1/m0,m1_m2}' > ${id%%.*}.nodup.pbc.qc;done &
```

Library complexity measures计算结果如下，...nodup.pbc.qc文件格式为：  
TotalReadPairs  
DistinctReadPairs   
OneReadPair  
TwoReadPairs  
NRF=Distinct/Total  
PBC1=OnePair/Distinct  
PBC2=OnePair/TwoPair

针对NRF、PBC1、PBC2这几个指标，ENCODE官网提供的标准如下:

![](https://blog-image-host.oss-cn-shanghai.aliyuncs.com/NRF-PBC1-PBC2.jpeg)

我们此次4个样本的分析结果显示这几个指标的值如下所示：

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/NRF PBC1 PBC2.JPG"/>

除了第一个样本（SRR12692092），其他样本计算结果显示NRF、PBC1、PBC2的值都非常完美，说明我们进行过滤和PCR去重的bam文件质量上没有问题，可以用于后续的分析。



### 6、插入片段质控检测
<table><tr><td bgcolor=GreenYellow>用于评估ATAC/Chip实验质量好坏的一个重要指标</td></tr></table>

```bash
#多个样本循环
picard_jar=/home/gongyuqi/miniconda3/envs/encode-atac-seq-pipeline/share/picard-2.20.7-0/picard.jar
nohup ls *.nodup.srt.name.bam | while read id; do java -Xmx6G -XX:ParallelGCThreads=1 -jar ${picard_jar} CollectInsertSizeMetrics INPUT=$id OUTPUT=${id%%.*}.INSERT.DATA H=${id%%.*}.INSERT.PLOT.pdf VERBOSITY=ERROR QUIET=TRUE USE_JDK_DEFLATER=TRUE USE_JDK_INFLATER=TRUE W=1000 STOP_AFTER=5000000; done &
```

成功的ATAC-seq实验会生成具有逐级递减和周期性的峰，对应无核小体区域（Nucleosome free region, NRF,100bp）,单核区域（mono-nucleosome,200bp），双核区域（400bp），三核区域（600bp）。

本次结果显示样本质量OK

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/SRR12692092.INSERT.PLOT.jpg" width="50%"><img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/SRR12692093.INSERT.PLOT.jpg" width="50%">

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/SRR12692098.INSERT.PLOT.jpg" width="50%"><img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/SRR12692099.INSERT.PLOT.jpg" width="50%">

### 7、TSS富集
**以BRM014-10uM_24h_wt样本为例，可视化peaks在基因组上的分布特征(TSS区域；TSS、gene-body、TSE区域)**

- both -R and -S can accept multiple files
- UCSC官网下载小鼠参考基因组注释文件，详情参考：http://www.bio-info-trainee.com/2494.html

```bash
#生成bw文件用于后续的computeMatrix
nohup ls *.nodup.bam |while read id
do
nohup bamCoverage --normalizeUsing CPM -b $id -o ${id%%.*}.nodup.bw
done &

#注释文件绝对路劲
refseq=/home/gongyuqi/ref/mm10/mouse.ucsc.refseq.bed

#TSS
##生成plotHeatmap需要的gz文件
nohup computeMatrix reference-point  --referencePoint TSS  -p 30 \
-b 3000 -a 3000 \
-R $refseq \
-S SRR12692098.nodup.bw SRR12692099.nodup.bw \
--skipZeros -o BRM014-10uM_24h_wt_TSS.gz \
--outFileSortedRegions BRM014-10uM_24h_wt_TSS.bed &
##可视化文件
nohup plotHeatmap -m BRM014-10uM_24h_wt_TSS.gz  -out BRM014-10uM_24h_wt_TSS_rainbow.png --colorMap rainbow &

#TSS_gene-body_TSE
##生成plotHeatmap需要的gz文件
nohup computeMatrix scale-regions -p 30 \
-a 3000 -b 3000 -m 5000 \
-R $refseq \
-S SRR12692098.nodup.bw SRR12692099.nodup.bw \
--skipZeros -o BRM014-10uM_24h_wt_TSS_gene0body_TSE.gz \
--outFileSortedRegions BRM014-10uM_24h_wt_TSS.bed &
##可视化文件
nohup plotHeatmap -m BRM014-10uM_24h_wt_TSS_genebody_TSE.gz  -out BRM014-10uM_24h_wt_TSS_genebody_TSE.png --colorMap rainbow &
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/BRM014-10uM_24h_wt_TSS_rainbow.png" width="50%"><img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/BRM014-10uM_24h_wt_TSS_genebody_TSE.png" width="50%">


## 五、生成tagAlign格式文件

### 1. Convert PE BAM to tagAlign
- 对于单端序列。直接用bed格式就可以；对于双端序列，推荐用bedpe格式。这两种格式都可以称之为tagAlign，可以作为macs的输入文件。
- tagAligen格式相比bam，文件大小会小很多，更加方便文件的读取。在转换得到tagAlign格式之后，我们就可以很容易的将坐标进行偏移

```bash
nohup ls *nodup.srt.name.bam|while read id; do bedtools bamtobed -bedpe -mate1 -i $id  | gzip -nc > ${id%%.*}.nodup.srt.name.bedpe.gz;done &
#含有chrM的染色体的TagAlign文件
nohup ls *.nodup.srt.name.bedpe.gz | while read id; do zcat $id | awk 'BEGIN{OFS="\t"}{printf "%s\t%s\t%s\tN\t1000\t%s\n%s\t%s\t%s\tN\t1000\t%s\n",$1,$2,$3,$9,$4,$5,$6,$10}' | gzip -nc > ${id%%.*}.nodup.srt.name.tagAlign.gz; done &
#去除chrM的染色体的TagAlign文件
nohup ls *nodup.srt.name.bedpe.gz|while read id; do zcat $id | grep -P -v "^chrM" | awk 'BEGIN{OFS="\t"}{printf "%s\t%s\t%s\tN\t1000\t%s\n%s\t%s\t%s\tN\t1000\t%s\n",$1,$2,$3,$9,$4,$5,$6,$10}' | gzip -nc > ${id%%.*}.nodup.nomit.srt.name.tagAlign.gz; done
```

### 2. Stand Cross Correlation analysis
<table><tr><td bgcolor=GreenYellow>用于评估ATAC/Chip实验质量好坏的一个重要指标</td></tr></table>

```bash
NREADS=25000000
nohup ls *.nodup.srt.name.bedpe.gz | while read id; do zcat $id | grep -v “chrM” | shuf -n ${NREADS} --random-source=<(openssl enc -aes-256-ctr -pass pass:$(zcat -f ${id%%.*}.nodup.srt.name.tagAlign.gz | wc -c) -nosalt </dev/zero 2>/dev/null) | awk 'BEGIN{OFS="\t"}{print $1,$2,$3,"N","1000",$9}' | gzip -nc > ${id%%.*}.nodup.nomit.srt.name.$((NREADS / 1000000)).tagAlign.gz; done &
#命令最终会生成交叉相关质量评估文件，*.cc.qc文件中会输出包含11列的信息，重点关注9-11列的信息，cc.plot.pdf文件相当于*.cc.qc文件的可视化
nohup ls *$((NREADS / 1000000)).tagAlign.gz | while read id; do Rscript $(which run_spp.R) -c=$id -p=10 -filtchr=chrM -savp=${id%%.*}.cc.plot.pdf -out=${id%%.*}.cc.qc; done &
#质控结果查看，主要看NSC,RSC,Quality tag三个值即输出文件的第9列，第10列，第11列。
ls *.cc.qc|while read id; do cat $id | awk '{print $9, "\t", $10, "\t", $11}';done 
```
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/cc.qc.JPG" width="50%">
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/SRR12692092.cc.plot.jpg" width="50%"><img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/SRR12692093.cc.plot.jpg" width="50%">

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/SRR12692098.cc.plot.jpg" width="50%"><img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/SRR12692099.cc.plot.jpg" width="50%">

- 质控结果解读
> Normalized strand cross-correlation coefficent (NSC)：NSC是最大交叉相关值除以背景交叉相关的比率(所有可能的链转移的最小交叉相关值)。NSC值越大表明富集效果越好，NSC值低于1.1表明较弱的富集，小于1表示无富集。NSC值稍微低于1.05，有较低的信噪比或很少的峰，这肯能是生物学真实现象，比如有的因子在特定组织类型中只有很少的结合位点；也可能确实是数据质量差。
> 
> Relative strand cross-correlation coefficient (RSC)：RSC是片段长度相关值减去背景相关值除以phantom-peak相关值减去背景相关值。RSC的最小值可能是0，表示无信号；富集好的实验RSC值大于1；低于1表示质量低。
> 
> QualityTag: Quality tag based on thresholded RSC (codes: -2:veryLow,-1:Low,0:Medium,1:High,2:veryHigh)
> 
> 查看交叉相关性质量评估结果，主要看NSC,RSC,Quality tag三个值，这三个值分别对应输出文件的第9列，第10列，第11列。



## 六、Call Peaks

### 1、去除线粒体基因的TagAlign格式文件进行shift操作，输入macs2软件去callpeak
```bash
smooth_window=150 # default
shiftsize=$(( -$smooth_window/2 ))
pval_thresh=0.01
nohup ls *nodup.nomit.srt.name.tagAlign.gz | while read id; \
do macs2 callpeak \
-t $id -f BED -n "${id%%.*}" -g mm -p $pval_thresh \
--shift $shiftsize --extsize $smooth_window --nomodel -B --SPMR --keep-dup all --call-summits; \
done &
```

### 2、去除ENCODE列入黑名单的区域
- 去除黑名单的bed文件，用于后续的peaks注释
```bash
BLACKLIST=/home/gongyuqi/project/ATAC/mm10.blacklist.bed.gz
#*_summits.bed为macs2软件callpeak的结果文件之一
nohup ls *_summits.bed | while read id; do bedtools intersect -a $id -b $BLACKLIST -v > ${id%%.*}_filt_blacklist.bed; done &
#查看过滤黑名单的区域前后的bed文件的peaks数
ls *summits.bed|while read id; do cat $id |wc -l >>summits.bed.txt;done
ls *summits_filt_blacklist.bed|while read id; do cat $id |wc -l >>summits_filt_blacklist.bed.txt;done
past summits.bed.txt summits_filt_blacklist.bed.txt 
```
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/before_after_blacklist-bed.JPG"/>





- 去除黑名单的narrowPeaks文件，用于后续的IDR评估
```bash
#使用IDR需要先对MACS2的结果文件narrowPeak根据-log10(p-value)进行排序,-log10(p-value)在第八列。
# Sort by Col8 in descending order and replace long peak names in Column 4 with Peak_<peakRank>
#*_peaks.narrowPeak为macs2软件callpeak的结果文件之一
NPEAKS=300000 
ls *_peaks.narrowPeak | while read id; do sort -k 8gr,8gr $id | awk 'BEGIN{OFS="\t"}{$4="Peak_"NR ; print $0}' | head -n ${NPEAKS} | gzip -nc > ${id%%_*}.narrowPeak.gz; done

BLACKLIST=../BLACKLIST/mm10.blacklist.bed.gz

#生成不压缩文件
ls *.narrowPeak.gz | while read id; do bedtools intersect -v -a $id -b ${BLACKLIST} | awk 'BEGIN{OFS="\t"} {if ($5>1000) $5=1000; print $0}' | grep -P 'chr[\dXY]+[ \t]'  > ${id%%.*}.narrowPeak.filt_blacklist; done
#生成压缩文件
#ls *.narrowPeak.gz | while read id; do bedtools intersect -v -a $id -b ${BLACKLIST} | awk 'BEGIN{OFS="\t"} {if ($5>1000) $5=1000; print $0}' | grep -P 'chr[\dXY]+[ \t]' | gzip -nc > ${id%%.*}.narrowPeak.filt_blacklist.gz; done
```

### 3、Irreproducibility Discovery Rate (IDR)评估
<table><tr><td bgcolor=GreenYellow>用于评估重复样本间peaks一致性的重要指标</td></tr></table>
首先生成narrowPeak_sample.txt文件，方便后续循环处理，生成文件内容如下：
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/narrowPeak for idr.JPG"/>

```bash
nohup cat narrowPeak_sample.txt | while read id
do
arr=(${id})
Rep1=${arr[0]}
Rep2=${arr[1]}
sample=${Rep1%%.*}_${Rep2%%.*}_idr
idr --samples $Rep1 $Rep2 --input-file-type narrowPeak -o $sample --plot  
done &
```

- DMSO_24h_wt （样本处理情况）
- SRR12692092.filt_blacklist.narrowPeak SRR12692093.filt_blacklist.narrowPeak
- 没有通过IDR阈值的显示为红色

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/SRR12692092_SRR12692093_idr_command.png"/>

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/SRR12692092_SRR12692093_idr.png"/>

- BRM014-10uM_24h_wt （样本处理情况）
- SRR12692098.filt_blacklist.narrowPeak SRR12692099.filt_blacklist.narrowPeak
- 没有通过IDR阈值的显示为红色

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/SRR12692098_SRR12692099_idr_command.png"/>
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/SRR12692098_SRR12692099_idr.png"/>

> IDR评估会同时考虑peaks间的overlap和富集倍数的一致性。通过IDR阈值（0.05）的占比越大，说明重复样本间peaks一致性越好。从idr的分析结果看，我们的测试数据还可以的呢。

**IDR评估相关参考资料：**  

[IDR]: https://www.jianshu.com/p/d8a7056b4294?utm_campaign=maleskine&utm_content=note&utm_medium=seo_notes&utm_source=recommendation           
[重复样本的处理——IDR][IDR]

### 4、Fraction of reads in peaks (FRiP)评估
<table><tr><td bgcolor=GreenYellow>反映样本富集效果好坏的评价指标</td></tr></table>

```bash 
#生成bed文件
nohup ls *.nodup.bam|while read id;do (bedtools bamtobed -i $id >${id%%.*}.nodup.bed) ;done &
#批量计算FRiP
ls *_summits_filt_blacklist.bed|while  read id;
do
echo $id
bed=${id%%_*}.nodup.bed 
Reads=$(bedtools intersect -a $bed -b $id |wc -l|awk '{print $1}')
totalReads=$(wc -l $bed|awk '{print $1}')
echo $Reads  $totalReads 
echo '==> FRiP value:' $(bc <<< "scale=2;100*$Reads/$totalReads")'%'
done
```
FRiP值在5%以上算比较好的。但也不绝对，这是个软阈值，可以作为一个参考。

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/FRiP.JPG"/>

**FRiP评估相关参考资料：**

https://www.jianshu.com/p/09e05bcd6981?utm_campaign=maleskine&utm_content=note&utm_medium=seo_notes&utm_source=recommendation



## 七、Peak annotation

### 1、Feature Distribution
```R
setwd("path to bed file")
library(ChIPpeakAnno)
library(TxDb.Mmusculus.UCSC.mm10.knownGene)
library(org.Mm.eg.db)
library(BiocInstaller)
library(ChIPseeker)
txdb <- TxDb.Mmusculus.UCSC.mm10.knownGene
promoter <- getPromoters(TxDb=txdb, upstream=3000, downstream=3000)
files = list(DMSO_24h_wt_rep1 = "SRR12692092_summits_filt_blacklist.bed",
             DMSO_24h_wt_rep2 = "SRR12692093_summits_filt_blacklist.bed",
             BRM014_10uM_24h_wt_rep1 = "SRR12692098_summits_filt_blacklist.bed",
             BRM014_10uM_24h_wt_rep2 = "SRR12692099_summits_filt_blacklist.bed")
#汇总所有样本
#plotAnnoBar和plotDistToTSS这两个柱状图都支持多个数据同时展示
peakAnnoList <- lapply(files, annotatePeak, 
                       TxDb=txdb,
                       tssRegion=c(-3000, 3000))
plotAnnoBar(peakAnnoList,title = "                        Feature Distribution")
plotDistToTSS(peakAnnoList,title = "                 Feature Distribution relative to TSS")
#例举单个样本
peakAnno <- annotatePeak(files[[1]],# 分别改成2或者3或者4即可，分别对应四个文件
                        tssRegion=c(-3000, 3000),
                        TxDb=txdb,
                        annoDb="org.Mm.eg.db")
plotAnnoPie(peakAnnoLipeakAnnost)
upsetplot(peakAnno, vennpie=TRUE)
```
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/Feature Distribution.jpeg" width="50%"><img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/Feature Distribution TSS.jpeg" width="50%">

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/plotAnnoPie.jpeg" width="50%"><img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/upsetplot(peakAnno, vennpie=TRUE).jpeg" width="50%">

### 2、查看peaks在全基因组上的分布
```R
#输入文件的准备
DMSO_24h_wt_rep1<-read.csv("SRR12692092_summits_filt_blacklist.csv")
DMSO_24h_wt_rep1<-DMSO_24h_wt_rep1[,-4]
DMSO_24h_wt_rep2<-read.csv("SRR12692093_summits_filt_blacklist.csv")
DMSO_24h_wt_rep2<-DMSO_24h_wt_rep2[,-4]
BRM014_10uM_24h_wt_rep1<-read.csv("SRR12692098_summits_filt_blacklist.csv")
BRM014_10uM_24h_wt_rep1<-BRM014_10uM_24h_wt_rep1[,-4]
BRM014_10uM_24h_wt_rep2<-read.csv("SRR12692099_summits_filt_blacklist.csv")
BRM014_10uM_24h_wt_rep2<-BRM014_10uM_24h_wt_rep2[,-4]

#以DMSO_24h_wt_rep1为例
set.seed(123)
circos.initializeWithIdeogram(plotType = c("axis", "labels"))
circos.track(ylim = c(0, 1), panel.fun = function(x, y) {
  chr = CELL_META$sector.index
  xlim = CELL_META$xlim
  ylim = CELL_META$ylim
  circos.rect(xlim[1], 0, xlim[2], 1)
}, track.height = 0.15, bg.border = NA, bg.col=rainbow(24))
text(0, 0, "DMSO_24h_wt_rep1", cex = 1.5)
circos.genomicDensity(DMSO_24h_wt_rep1, col = c("#000080"), track.height = 0.2)
circos.clear()
```
看到这样的结果，第一反应就是————为什么两种处理情况下染色体开放程度那么像！？难道我代码有问题！？经过反复检查验证（将一个样本chr1上的peaks都删掉，再次运行上述代码，就会发现显著的改变），可以确定分析上是没有问题的。这两种处理导致的差异可能不是很显著。再加上20万+的peaks放在这个小小的circos图上展示，有些差异会被掩盖掉。就如在做TSS富集分析的时候，单独看TSS前后3Kb区域，可以看到有两个峰，但在看TSS-genebody-TSE区域是，TSS处相对微弱的那个峰就被掩盖掉了。

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/DMSO_24h_wt_rep1.jpeg" width="50%"><img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/DMSO_24h_wt_rep2.jpeg" width="50%">
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/BRM014_10uM_24h_wt_rep1.jpeg" width="50%"><img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/BRM014_10uM_24h_wt_rep2.jpeg" width="50%">

### 3、拿到每个样本中peaks对应得基因名

这一步非常重要，拿到基因名就可以根据课题需要进行差异分析等
```R
#以DMSO_24h_wt样本为例
#replicate 1
peakAnno_DMSO_24h_wt_rep1 <- annotatePeak(files[[1]],
                                          tssRegion=c(-3000, 3000),
                                          TxDb=txdb,
                                          annoDb="org.Mm.eg.db")
genelist_DMSO_24h_wt_rep1_uniqe<-as.data.frame(unique(peakAnno_DMSO_24h_wt_rep1@anno@elementMetadata@listData[["SYMBOL"]]))
colnames(genelist_DMSO_24h_wt_rep1_uniqe)<-"symbol"
#replicate 2
peakAnno_DMSO_24h_wt_rep2 <- annotatePeak(files[[2]],
                                          tssRegion=c(-3000, 3000),
                                          TxDb=txdb,
                                          annoDb="org.Mm.eg.db")
genelist_DMSO_24h_wt_rep2_uniqe<-as.data.frame(unique(peakAnno_DMSO_24h_wt_rep2@anno@elementMetadata@listData[["SYMBOL"]]))
colnames(genelist_DMSO_24h_wt_rep2_uniqe)<-"symbol"
#重复样本间共同的开放基因
venn.diagram(
  x=list(
    DMSO_24h_wt_rep1=genelist_DMSO_24h_wt_rep1_uniqe$symbol,
    DMSO_24h_wt_rep2=genelist_DMSO_24h_wt_rep2_uniqe$symbol
  ),
  filename = "DMSO_24h_wt.png",
  lty="dotted",
  lwd=3,
  col="transparent",
  fill=c("darkorchid1","cornflowerblue"),
  alpha=0.5,
  
  label.col=c("darkorchid1","white","darkblue") ,
  cex=1,
  fontfamily="serif",
  fontface="bold",
  cat.default.pos="text",
  cat.col=c("darkorchid1","darkblue"),
  cat.cex=0.6,
  cat.fontfamily="serif",
  cat.dist=c(0.3,0.3),
  cat.pos=0
)

#查看各组内样本间的overlapping reads：DMSO_24h_wt， BRM014_10uM_24h_wt；
#以及组间peaks的异同情况：DMSO_24h_wt vs. BRM014_10uM_24h_wt
#代码类似上面的，就不一一展示了
```
从下图可以看出，不管是组间还是组内，差异的peaks数目都不是很多了，这一点也验证了我们上面做的再全基因组范围内查看peaks的分布结果。

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/DMSO_24h_wt.png" width="50%"><img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/BRM014_10uM_24h_wt.png" width="50%">
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/DMSO_BRM014.png" width="50%">


