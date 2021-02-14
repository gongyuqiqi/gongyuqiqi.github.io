---
title: 特定版本R包下载及其报错
date: 2020-02-16 16:30:23
tags: R
---

Error: Failed to install 'unknown package' from URL:  
(converted from warning) installation of package ‘dtw’ had non-zero exit status

<!--more-->

# 特定版本R包下载及其报错  

## 下载Seurat 2.×.× 

首先查看版本信息：https://github.com/satijalab/seurat/releases

```R
require(devtools)
install_version("Seurat",version = "2.3.3")
```

**报错如下**

```R
Error: Failed to install 'unknown package' from URL:
(converted from warning) installation of package ‘dtw’ had non-zero exit status
```

显示R包`dtw`退出码非零，即不正常退出，思考再三，是不是因为这个Seurat依赖的R包没有存在并且下载不成功导致的呢？   
尝试单独下载R包`dtw `

```R
install.packages("dtw")
```

```R
package ‘dtw’ successfully unpacked and MD5 sums checked
The downloaded binary packages are in ...
```

## 再次下载Seurat 2.×.×

```R
require(devtools)
install_version("Seurat",version = "2.3.3")
```

**再次得到报错信息**

```R
Error: Failed to install 'unknown package' from URL:
(converted from warning) installation of package ‘doSNOW’ had non-zero exit status
```

- 此次报错和上次类似，但变成另一个R包退出码非零，说明上次的排错思路可能是对的。
- 于是后面每次出现类似的报错就单独下载相应的R包，在尝试下载Seurat 2.3.3。

**第N次得到报错信息**

1. 用以下方法试图下载Seurat 2.3.3依赖的R包`SDMTool`

```R
#方法1
install.packages("SDMTools")
#方法2
BiocManager::install("SDMTools")
#方法3
source("https://bioconductor.org/biocLite.R")
biocLite("SDMTools")
```
2. 遗憾的是，报错了，报错信息都是以下内容

😭<font color=blue>你时常感概，国内的网怎么老是不能通向世界呢😭</font>

```R
package ‘SDMTools’ is not available (for R version 3.5.2)
```

1. 所以只好手动下载`SDMTools`,手动安装  

- 首先登陆[CRAN官网](https://cran.r-project.org/)，点击进入**Download R for Windows**（这个视不同系统而定）
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/CRANwedsite.JPG"/>

- 点击contrib
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/CRANcontrib.JPG"/>

- 点击相应的R版本，进入R包界面，`ctrl+F`搜索所需要的R包，下载到本地
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/DMTools.JPG"/>

- 进入Rstudio,点击Tools,点击install packages...或者直接点击environment下的install
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/unautoinstall.JPG"/>   
然后点击install即可

## 当你再次下载Seurat 2.3.3时，遇上第N+1次报错
当你解决了上诉报错问题后，你还会收到各种各样的报错，比如缺少这个包，缺少那个包。你就当体验一次手动解决R包之间的依赖关系好了。直到不再提示缺少什么R包。

## 再再再次次次下载Seurat 2.3.3

```R
require(devtools)
install_version("Seurat",version = "2.3.3")
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/seuratdone.JPG"/>

**成功！！！**

## 加载Seurat并查看相关信息

```R
library(Seurat)
sessionInfo()
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/seuratinfo.JPG"/>
