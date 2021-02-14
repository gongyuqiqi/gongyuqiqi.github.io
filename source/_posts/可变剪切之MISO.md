---
title: MISO
date: 2021-01-09 14:05:52
tags: NGS
---

<!--more-->


# 可变剪切分析之MISO
MISO软件原文档：https://miso.readthedocs.io/en/fastmiso/index.html#installing-fastmiso
## 数据下载

- 写一个bash脚本下载数据  
一次性下载所有的.fastq.gz文件，本次实验数据是单端测序数据
```bash
#！/bin/bash
dir=/home/gongyuqi/.aspera/connect/etc/asperaweb_id_dsa.openssh
for id in  {84,78,76,72,82,80,74,68} 
do 
       nohup ascp -QT -l 300m -P33001 -i $dsa era-fasp@fasp.sra.ebi.ac.uk:/vol1/fastq/SRR974/SRR9749$id/SRR9749$id.fastq.gz . &
done 
```
- 执行脚本

```bash
chmod +x download.sh
nohup ./download.sh &
```
## 比对

参考资料：http://www.360doc.com/content/18/0714/20/19913717_770401548.shtml

> - tophat进行转录组的比对时，输出的是bam（二进制的sam）文件。
> - tophat依赖bowtie2（bowtie1也可以），参考基因组索引用bowtie2-build进行构建。
> - 如果FASTA格式的基因组与索引文件不在同一个文件夹下面，tophat会在运行中自动生成，所以记得把参考基因组和索引文件放在同一个文件夹下面，以减少系统运行时的高消耗。
> - 如果GTF或者GFF格式的基因组注释文件存在， tophat会优先比对到注释文件的转录组上。最好事先准备好转录组索引以节约后续的比对时间。
> - 如果没有事先准备好转录组索引，tophat会在运行过程中生成转录组索引，这样就比较耗时耗力了。


**<table><tr><td bgcolor=GreenYellow>**参考基因组索引构建、转录组索引构建**</td></tr></table>**

hg19参考基因组gtf文件下载链接：https://www.gencodegenes.org/human/release_36lift37.html

```bash
#建立hg19参考基因组索引文件
nohup time bowtie2-build path/to/hg19.fa ./hg19 > hg19.bowtie_index.log 2>&1 &
#建立hg19转录组索引文件
#../index/bowtie/hg19为上一步构建的hg19参考基因组索引文件的相对路径及前缀
nohup tophat -G ../gencode.v36lift37.annotation.gtf --transcriptome-index=hg19.tr ../index/bowtie/hg19 &
```


**<table><tr><td bgcolor=GreenYellow>**参考基因组的比对**</td></tr></table>** 

##### 比对方法一
```bash
#以其中一个样本为例，走一波比对转录组流程。
#理论上可以节约时间，但是实际跑起来非常耗时！
#1G左右得样本跑了4个小时都没有跑完！！！不知道问题出在那儿，但是还是果断kill掉
tr_index=/home/gongyuqi/ref/hg19/transcriptome-index/hg19.tr
gene_index=~/ref/hg19/index/bowtie/hg19
tophat2 -o ../aligndata/SRR974980.out --transcriptome-index=$tr -p 30 --phred64-quals $gene_index SRR974980.fastq.gz
```

##### 比对方法二
- 写一个比对的bash脚本文件（align.txt），如下
```bash 
#!/bin/bash
ls *.gz|while read id
do
nohup tophat -G ~/ref/hg19/gencode.v36lift37.annotation.gtf -p 30 -o ../aligndata/$id.out ~/ref/hg19/index/bowtie/hg19 $id &
done 
```
- 执行这个脚本，然后去睡觉，早上醒来看看比对得结果如何~
```bash
nohup bash align.txt &
```
运行结果
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/align1.JPG"/>


每个文件夹打开结果如下
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/align2.JPG"/>

大概了解一下样本的运行日志
>注意reads的长度——36。miso后续分析中需要知道这个值  
>注意其中有一个error,至于为什么有这个报错，先不追究  
>注意比对结果的那个统计文件align_summary.txt,查看这个文件发现各个样本比对情况都挺好的。这就是为什么这里暂时不追究那个报错  
>最后会统计样本比对时长
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/align4.JPG"/>

3、bam文件排序及索引文件建立

```bash
ls *.bam|while read id; do samtools sort -@ 30 -m 8G -O bam $id -o ${id%%.*}.accepted_hits.sorted.bam;done
ls *sorted.bam|while read id; do nohup samtools index -@ 30 $id & done
```

## 运行MISO

**<table><tr><td bgcolor=GreenYellow>**下载hg19的GFF文件并构建索引**</td></tr></table>**

具体下载网页及详情：https://miso.readthedocs.io/en/fastmiso/annotation.html
```bash
cd /home/gongyuqi/project/AS/annotation/human/hg19
wget hollywood.mit.edu/burgelab/miso/annotations/ver2/miso_annotations_hg19_v2.zip
unzip miso_annotations_hg19_v2.zip
```
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/unzipgff.JPG"/>

这里我们以外显子的gff3为例构建索引

```bash
index_gff --index SE.hg19.gff3 ./index
```
结果如下
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/ESgff3index.JPG"/>


**<table><tr><td bgcolor=GreenYellow>**以单端测序样本为例运行miso**</td></tr></table>**

这一步生成的结果是很关键的，会计算alternative splicing events的PSI值。而这个在最后可视化的阶段会以posterior distributions over Ψ(PSI)的形式呈现——即sashimi_plot图最右边一列的柱状图。

- 写一个miso_run的脚本，内容如下
```bash
#!/bin/bash
cd /home/gongyuqi/project/AS/aligndata/accpted_bam
index_db=/home/gongyuqi/project/AS/annotation/human/hg19/index/
out_dir=/home/gongyuqi/project/AS/miso_step1
ls *.sorted.bam|while read id
do
nohup miso --run $index_db $id --output-dir $out_dir/${id%%.*} --read-len 36 &
done
```

- 运行脚本
```bash
nohup ./miso_run.sh &
```

举例查看其中一个样本的运行结果

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/miso_run_out.JPG"/>


<table><tr><td bgcolor=GreenYellow>**样本间比较**</td></tr></table>

这个软件最大的问题在于只能进行两个样本的比较，这一点就很气！！！  
气归气，还是先运行一下，至少先把流程走完，先学起来。
```bash
compare_miso --compare-samples SRR974978/ SRR974982/ comparison/
```


**<table><tr><td bgcolor=GreenYellow>**过滤**</td></tr></table>**

设置各种参数的阈值，目的是筛选差异显著的alternative splicing events。
```bash
filter_events \
--filter  SRR974978_vs_SRR974982.miso_bf \
--num-inc 1 \
--num-exc 1 \
--num-sum-inc-exc 10 \
--delta-psi 0.20 \
--bayes-factor 10 \
--output-dir ./filter_out
```
过滤前后.misoz_bf文件中的alternative splicing events有明显减少
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/filter.JPG"/>


**<table><tr><td bgcolor=GreenYellow>**可视化**</td></tr></table>**

```bash
index_db=/home/gongyuqi/project/AS/annotation/human/hg19/index/
#下面做演示的剪切事件是从上一步过滤后的.miso_bf.filtered文件中提取的一个
sashimi_plot \
--plot-event "chr20:1093906:1094052:+@chr20:1099396:1099545:+@chr20:1106141:1106293:+" \
$index_db \
sashimi_plot_settings.txt  \
--output-dir ./sashimi_plot
```


其中sashimi_plot_settings.txt是配置文件，内容如下。当然，也可以根据自己的作图需要修改相应的参数
```bash
[data]
# directory where BAM files are
bam_prefix = /home/gongyuqi/project/AS/aligndata/accpted_bam/
# directory where MISO output is
miso_prefix = /home/gongyuqi/project/AS/miso_step1

bam_files = [
    "SRR974978.accepted_hits.sorted.bam",
    "SRR974984.accepted_hits.sorted.bam",
    "SRR974968.accepted_hits.sorted.bam",
    "SRR974972.accepted_hits.sorted.bam",
    "SRR974982.accepted_hits.sorted.bam",
    "SRR974980.accepted_hits.sorted.bam",
    "SRR974974.accepted_hits.sorted.bam",
    "SRR974976.accepted_hits.sorted.bam"]

miso_files = [
    "SRR974978",
    "SRR974984",
    "SRR974968",
    "SRR974972",
    "SRR974982",
    "SRR974980",
    "SRR974974",
    "SRR974976"]

[plotting]
# Dimensions of figure to be plotted (in inches)
fig_width = 4
fig_height = 10
# Factor to scale down introns and exons by
intron_scale = 30
exon_scale = 4
# Whether to use a log scale or not when plotting
logged = False
font_size = 6

# Max y-axis
ymax = 150

# Whether to plot posterior distributions inferred by MISO
show_posteriors = True

# Whether to show posterior distributions as bar summaries
bar_posteriors = False

# Whether to plot the number of reads in each junction
number_junctions = True

resolution = .5
posterior_bins = 40
gene_posterior_ratio = 5

# List of colors for read denisites of each sample
colors = ["#CC0011",
          "#CC0011",
          "#CC0011",
          "#CC0011",
          "#FF8800",
          "#FF8800",
          "#FF8800",
          "#FF8800"]

# Number of mapped reads in each sample
# (Used to normalize the read density for RPKM calculation)
coverages = [27232306,
             16799279,
             31298963,
             26308550,
             27067722,
             17372484,
             26095030,
             26980460]

# Bar color for Bayes factor distribution
# plots (--plot-bf-dist)
# Paint them blue
bar_color = "b"

# Bayes factors thresholds to use for --plot-bf-dist
bf_thresholds = [0, 1, 2, 5, 10, 20]
```

可视化结果如下  
因为我们选取的剪切事件是SRR974978和SRR974982之间的差异剪切事件，所以在多个样本间趋势不是很明显。但这里我们先学个方法。
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/sashimi_plot1.JPG"/>


