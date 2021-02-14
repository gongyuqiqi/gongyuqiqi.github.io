---
title: Markdown学习教程
date: 2020-02-01 16:30:23
tags: notes
---

基础教程，随便写写

<!--more-->

# Markdown学习教程  

![](https://ftp.bmp.ovh/imgs/2020/02/993e8d3010da3e06.jpg)  
>Markdown语言在2004年由John Gruber创建，是以一种轻量级标记语言，它允许人们使用易读写的纯文本格式编写文档。Markdown编写的文档可以导出HTML、Word、PDF、Epub等多种格式的文档，编写的文档后缀为`.md`，`.markdown`。       

>本学习教程在**Windows**下使用**visual studio code**编辑器讲解Markdown的语法。     
Visual Studio Code官网：https://aka.ms/win32-x64-user-stable  
Visual Studio Code(for windows)下载地址：https://aka.ms/win32-x64-user-stable

## Markdown 标题  
**1、使用#号可表示1-6级标题，一级标题对应一个#号，六级标题对应六个#号。**
```
# 一级标题
## 二级标题
### 三级标题
#### 四级标题
##### 五级标题
###### 六级标题
```   

# 一级标题
## 二级标题
### 三级标题
#### 四级标题
##### 五级标题
###### 六级标题    
**2、使用=和-标记一级标题和二级标题**     
```
一级标题
=======   
二级标题
-------
```
一级标题
=======   
二级标题
-------

## Markdown 段落  
**Markdown段落的换行是使用两个以上空格加上回车。**   
**Markdown段落的换行也可以使用一个空行来开始新的一行**   

## Markdown 列表
Markdown支持无序列表和有序列表。

**无序列表使用星号(*)、加号(+)、或是减号(-)作为列表标记。** 符号后面一定要空一格。 
```
* 第一项
* 第二项
* 第三项  
+ 第一项
+ 第二项
+ 第三项  
- 第一项
- 第二项
- 第三项
```    
* 第一项
* 第二项
* 第三项  
+ 第一项
+ 第二项
+ 第三项  
- 第一项
- 第二项
- 第三项  

**有序列表使用数字并加上`.`号表示。**  
```
1. 第一项
2. 第二项
3. 第三项
```  
1. 第一项  
2. 第二项
3. 第三项         


**列表嵌套**(注意使用tab键)
 ```
 1. 第一项
    - 第一项嵌套的第一个元素
    - 第一项嵌套的第二个元素
 2. 第二项  
    - 第二项嵌套的第一个元素
    - 第二项嵌套的第二个元素 
 ```
1. 第一项  
   - 第一项嵌套的第一个元素
   - 第一项嵌套的第二个元素  
2. 第二项   
   - 第二项嵌套的第一个元素
   - 第二项嵌套的第二个元素  

  

## Markdown 区块  
Markdown区块引用是在段落开头使用`>`符号，然后后面紧跟一个**空格**符号(为什么亲测`>`符号后面跟不跟空格都没关系呢)。  
```
> 区块引用
> 区块引用
> 区块引用
```  
> 区块引用  
> 区块引用  
> 区块引用    

区块也可以嵌套，一个`>`符号是最外层，两个`>`符号是第一层嵌套，以此类推。  
```
> 最外层
>> 第一层嵌套
>>> 第二层嵌套
```   
> 最外层   
>> 第一层嵌套   
>>> 第二层嵌套   

区块中使用列表  
```
> 区块中使用列表
> 1. 第一项
> 2. 第二项
> + 第一项
> + 第二项
> + 第三项
```   
> 区块中使用列表  
> 1. 第一项  
> 2. 第二项  
> + 第一项  
> + 第二项  
> + 第三项  

列表中使用区块  
```
* 第一项   
    > 第一项的第一步 
    > 第一项的第二部  
* 第二项
```   
* 第一项    
    > 第一项的第一步   
    > 第一项的第二部   
* 第二项  
    > 第二项的第一步
    > 第二项的第二部   

## Markdown 代码
如果是段落上的一个函数或者片段的代码可以用引号把它包起来(``).
```
`printf()` 函数
```   
`printf()`函数  

**代码区块**     
代码区块使用4个空格或者一个制表符(tab键)。   
>      mkdir test  
>      touch test.txt  
>      echo "This is a test"   
  
    mkdir test  
    touch test.txt  
    echo "This is a test"  

代码区块也可以用两个` ``` `包裹一段代码，并指定一种语言（也可以不指定）  
*```*   
mkdir test  
touch test.txt  
echo "This is a test"  

*```* 
```
mkdir test
touch test.txt
echo "This is a test"
```   
  


  
 

## Markdown 链接      
**显示关键词，指向一个链接**
```
[关键词](链接地址 "（可选）添加一个标题")   
[百度](https://www.baidu.com/)    
```   
[百度](https://www.baidu.com/)  

**展示链接地址**
``` 
<链接地址>  
或者
链接地址
```     
<www.baidu.com>       
https://www.baidu.com/   

**引用**  
如需多次添加某个链接，可以先给链接取个名字，下次再用到的时候就可以直接使用这个链接的名字，不用一直记住链接地址。  
```      
给这个链接取个名字：
[李老师]: https://lzqblog.top/2020-01-31/%E4%B8%AA%E4%BA%BA%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA%E7%AE%80%E6%98%8E%E6%95%99%E7%A8%8B-%E7%89%B9%E4%BE%9B%E7%89%88/#more    
使用这个链接名字       
[一篇美好小文章][李老师]
```    
[李老师]: https://lzqblog.top/2020-01-31/%E4%B8%AA%E4%BA%BA%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA%E7%AE%80%E6%98%8E%E6%95%99%E7%A8%8B-%E7%89%B9%E4%BE%9B%E7%89%88/#more     
重要的事情说三遍，嘿嘿  
[一篇美好小文章][李老师]  
[一篇美好小文章][李老师]  
[一篇美好小文章][李老师]

**将所有链接放文末**
```   
I get 10 times more traffic from [Google] [1] than from
[Yahoo] [2] or [MSN] [3].

  [1]: http://google.com/        "Google"
  [2]: http://search.yahoo.com/  "Yahoo Search"
  [3]: http://search.msn.com/    "MSN Search"
```
I get 10 times more traffic from [Google] [1] than from
[Yahoo] [2] or [MSN] [3].

  [1]: http://google.com/        "Google"
  [2]: http://search.yahoo.com/  "Yahoo Search"
  [3]: http://search.msn.com/    "MSN Search"
  

## 插入图片   
```
![]()
![图片名称](图片地址 "option title")
```  
+ 开头一个感叹号！
+ 接着一个方括号，里面放上图片的替代文字，可以不用写。
+ 接着一个普通括号，里面放上图片地址。"option title":鼠标悬置于图片上会出现的标题文字，可以不写。 

目前有很多免费的在线图床网站。这里使用[ImgURL](https://imgurl.org/)在线工具,上传要插入的图片，网页会自动生成URL等地址，如下图所示。 

![](https://ftp.bmp.ovh/imgs/2020/02/160e2425ef881f4f.jpg)

用上诉方法插图一张萌萌的图片
```
![](https://ftp.bmp.ovh/imgs/2020/02/4c9db7d1991a2aa7.jpg)
``` 
![](https://ftp.bmp.ovh/imgs/2020/02/4c9db7d1991a2aa7.jpg)  


 
如果是firefox浏览器，右键鼠标点击复制图像地址，有的地址很长很长很长，但也不是每次复制的图像地址都能成功，所以这个方法比较悬。当然不同浏览器会有一定的区别。 
```
![](这里的地址有时候会非常长很长十分长......)
```   
但是呢，我们家**可爱的李老师**用阿里oss搭建了自己的图床，还非常<font color=pink>sweet</font>的教我用。所以呢，我现在就不用这么麻烦得用上面得方法啦！
   
**设置图片大小**    
```  
设置图片大小方法一
<img src="图片地址" width="30%" height="30%">  
原图大小  
![](https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/pooh.gif)  
改变图片大小  
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/pooh.gif" width="20%" height="20%">
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/pooh.gif" width="15%" height="15%"> 
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/pooh.gif" width="10%" height="20%">  

设置图片大小方法二
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/pooh.gif">   
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/pooh.gif"  width="200" height="200">   
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/pooh.gif"  width="150" height="150">  
（方法二的结果就不展示了）
```  
原图大小  
![](https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/pooh.gif)    
调整后大小   
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/pooh.gif" width="20%" height="20%">
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/pooh.gif" width="15%" height="10%"> 
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/pooh.gif" width="10%" height="15%">   
  

```   
图片位置设置  
<div align=left><img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/pooh.gif"></div>
<div align=center><img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/pooh.gif"></div>
<div align=right><img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/pooh.gif"></div>
```   
<div align=left><img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/pooh.gif"></div>
<div align=center><img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/pooh.gif"></div>
<div align=right><img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/pooh.gif"></div>


## Markdown 表格   
Markdown制作表格使用`|`来分隔不同的单元格，使用`-`来分隔表头和其他行。   
```  
| 表头 | 表头 |   
| ---- | ---- |   
| 单元格 | 单元格 |   
| 单元格 | 单元格 |
```

| 表头 | 表头 |   
| ---- | ---- |   
| 单元格 | 单元格 |   
| 单元格 | 单元格 |   

Markdown还可以设置表格的对其方式  
`-:`设置内容和标题栏居右对齐。  
`:-`设置内容和标题栏居左对齐。  
`:-:`设置内容和标题栏居中对齐。  
```  
| 左对齐 | 居中 | 右对齐|  
| :---- | :----: | ----: |   
| 单元格为左对齐 | 单元格居中 | 单元格右对齐 |   
| 单元格为左对齐 | 单元格居中 | 单元格右对齐 |
```  
| 左对齐 | 居中 | 右对齐|  
| :---- | :----: | ----: |   
| 单元格为左对齐 | 单元格居中 | 单元格右对齐 |   
| 单元格为左对齐 | 单元格居中 | 单元格右对齐 |   
## Markdown 初级技巧  
**斜体、粗体**
```   
*single asterisks*

_single underscores_

**double asterisks**

__double underscores__  

<em>single asterisks</em>

<em>single underscores</em>

<strong>double asterisks</strong>

<strong>double underscores</strong>
```

*single asterisks*

_single underscores_

**double asterisks**

__double underscores__  

<em>single asterisks</em>

<em>single underscores</em>

<strong>double asterisks</strong>

<strong>double underscores</strong>   

**删除线**
```   
~~这是要删除的内容~~  
```  
~~这是要删除的内容~~    

**字体添加下划线**   
```  
<u>需要添加下划线的字体</u>
```    
<u>需要添加下划线的字体</u>





## Markdown 高级技巧  
**转义**  
Markdown使用很多特殊符号来表示特定的意义，如果需要显示特定的符号则需要使用转义字符，Markdown使用反斜杠转义字符。  
```   
**文本加粗**  
\*\*正常显示星号\*\*
```    
**文本加粗**  
\*\*正常显示星号\*\*  

Markdown支持以下这些符号前面加上反斜杠来帮助插入普通的符号。  
\ 反斜线      
` 反引号  
\* 星号  
\- 下划线  
{} 花括号  
[] 方括号  
() 小括号  
\# 井号键   
\+ 加号  
\- 减号  
. 英文句点  
! 感叹号   

**更改字体、大小、颜色**   
```   
字体   
<font face="字体名称">显示的内容</font>
<font face="黑体">黑体</font>   
<font face="Times New Roman">Times New Roman</font>   
<font face="Arial">Arial</font>   
颜色   
<font color=red>红色</font>  
<font color=orange>橙色</font>  
<font color=yellow>黄色</font>   
$\color{#FF0000}{红}$ $\color{#FF7D00}{橙}$ $\color{#FF0000}{黄}$ $\color{#00FF00}{绿}$  $\color{#0000FF}{蓝}$ $\color{#00FFFF}{靛}$ $\color{#FF00FF}{紫}$
尺寸（可能的值：从 1 到 7 的数字。浏览器默认值是 3）  
<font size=1>字体尺寸大小为1</font>  
<font size=2>字体尺寸大小为2</font>  
<font size=3>字体尺寸大小为3</font>
```        
<font face="黑体">黑体</font>  
<font face="Times New Roman">Times New Roman</font>   
<font face="Arial">Arial</font>  
<font color=red>红色</font>  
<font color=orange>橙色</font>  
<font color=yellow>黄色</font>   
$\color{#FF0000}{红}$ $\color{#FF7D00}{橙}$ $\color{#FF0000}{黄}$ $\color{#00FF00}{绿}$  $\color{#0000FF}{蓝}$ $\color{#00FFFF}{靛}$ $\color{#FF00FF}{紫}$  
<font size=1>字体尺寸大小为1</font>  
<font size=2>字体尺寸大小为2</font>  
<font size=3>字体尺寸大小为3</font>   

**为文字添加背景色**   

借助table,tr,td等表格标签的bgcolor属性来实现背景色。这里对于文字背景色的设置，只是讲那一整行看作一个表格，更改了那个格子的背景色。   
```
<table><tr><td bgcolor=GreenYellow>GreenYellow</td></tr></table>
```
<table><tr><td bgcolor=GreenYellow>GreenYellow</td></tr></table>


**HTML标签'< span>'可同时实现更改字体、大小、颜色等要求** 
```   
<span style="color:Hotpink;font-family:Impact;font-style:Arial;cursor:crosshair;font-size:20px">Impact font Hotpink Arial 20px with crosshair pointer</span> <u><font size=1>鼠标放上去试一试</font></u>  

<span style="color:Hotpink; font-family:Georgia; font-size:20px;">Mr Li is so cute.</span>    

<span style="color:Hotpink; font-family:Georgia;cursor:crosshair; font-size:20px;">Mr Li is so cute.</span> <u><font size=1>鼠标放上去试一试</font></u>
```
<span style="color:Hotpink;font-family:Impact;font-style:Arial;cursor:crosshair;font-size:20px">Impact font Hotpink Arial 20px with crosshair pointer</span> <u><font size=1>鼠标放上去试一试</font></u>  

<span style="color:Hotpink; font-family:Georgia; font-size:20px;">Mr Li is so cute.</span>    

<span style="color:Hotpink; font-family:Georgia;cursor:crosshair; font-size:20px;">Mr Li is so cute.</span> <u><font size=1>鼠标放上去试一试</font></u>



**复选框**   
使用`- [ ]`和`- [×]`语法可以创建复选框  
```
- [x] Markdown  
- [ ] JavaScript  
+ [x] Markdown  
+ [ ] JavaScript  
* [x] Markdown  
* [ ] JavaScript    
```
- [x] Markdown  
- [ ] JavaScript     
+ [x] Markdown  
+ [ ] JavaScript  
* [x] Markdown  
* [ ] JavaScript     



没有的以后想到再说吧~~~    
#### 参考资料   
**https://lzqblog.top/2018-11-24/Markdown-frequently-used-syntax/**    
https://www.runoob.com/markdown/md-tutorial.html   
https://daringfireball.net/projects/markdown/syntax#html 
https://blog.csdn.net/lewky_liu/article/details/85010827   
https://blog.csdn.net/qq_43731019/article/details/89385836   
https://www.jianshu.com/p/cdd313eebfd9     
https://blog.csdn.net/heimu24/article/details/81189700
    