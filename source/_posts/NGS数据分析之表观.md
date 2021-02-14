---
title: NGS数据分析之表观
date: 2020-05-14 11:16:18
tags: NGS
---

 NGS数据分析之表观中的一些注意事项

<!--more-->

## 一、参考基因组及注释文件的准备

以果蝇为例

(1)下载果蝇参考基因组

打开谷歌(**用谷歌用谷歌**)，输入：drosophila_melanogaster ftp ensembl

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/1-果蝇基因组.JPG"/>

点击红色箭头所指方向进入正确的下载网页（这个需要自己悟，总是要多试的）

①
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/2-果蝇基因组.JPG"/>

②
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/3-果蝇基因组.JPG"/>

(2)下载参考基因组索引

打开谷歌，输入：drosophila_melanogaster ftp hisat2

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/4-果蝇基因组索引.JPG"/>

点击红色箭头所指方向进入正确的下载网页（这个需要自己悟，总是要多试的）

①
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/5-果蝇基因组索引.JPG"/>

②
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/6-果蝇基因组索引.JPG"/>


<font color=blue>**总结**：参考基因组下载方式很多，版本也不唯一，重要的是知道自己需要那个版本，这一点需要悟......就参考基因组索引而言，不同比对软件需要的索引文件不一样，可以自己构建，也可以网页下载，推荐按直接下载。</font>