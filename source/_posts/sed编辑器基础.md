---
title: sed编辑器基础
date: 2020-03-07 18:41:22
tags: Linux
---

sed编辑器更多的命令和格式：  
更多的替换选项、使用地址、删除行、插入和附加文本、修改行、转换命令、打印、使用sed处理文件

<!--more-->

# sed编辑器基础

## 更多的替换选项

### 1、替换标记  

替换命令在替换多行中的文本时能正常工作，但默认情况下它只替换每行中出现的第一处。要让替换命令能够替换一行中不同地方出现的文本必须使用替换标记（substitution flag）。

s/pattern/replacement/flags

有4种可用的替换标记： 
- 数字，表明新文本将替换第几处模式匹配的地方 
- g，表明新文本将会替换所有匹配的文本 
- p，表明原先行的内容要打印出来 
- w file，将替换的结果写到文件中

```bash
#默认情况下sed只替换每行中出现的第一处。
sed 's/test/trial/' sed1.txt
#sed替换每行中第二初匹配的地方。
sed 's/test/trial/2' sed1.txt
#替换所有匹配的文本。
sed 's/test/trial/g' sed1.txt
#p替换标记会输出修改过的行。
#sed命令的正常输出就是STANDOUT。
sed 's/test/trial/p' sed1.txt
#p替换标记会输出修改过的行。
sed 's/test/trial/p' sed2.txt
#-n选项将禁止sed编辑器输出。但p替换标记会输出修改过的行。将二者配合使用的效果就是只输出被替换命令修改过的行。
sed -n 's/test/trial/p' sed2.txt
#sed编辑器的正常输出是在STDOUT中，只有那些包含匹配模式的行才会保存在指定的输出文件中。
sed 's/test/trial/w test.txt' sed2.txt
cat test.txt
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/sed1.jpg"/>

### 2、替换字符
有时你会在文本字符串中遇到一些不太方便在替换模式中使用的字符。Linux中一个常见的 例子就是正斜线（/），由于正斜线通常用作字符串分隔符，因而如果它出现在了模式文本中的话，必须用反斜线来转义。

要解决这个问题，sed编辑器允许选择其他字符来作为替换命令中的字符串分隔符（与需要替换的转义字符不一致的一般都可以）。
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/sed1.2.jpg"/>


## 使用地址

默认情况下，在sed编辑器中使用的命令会作用于文本数据的所有行。如果只想将命令作用 于特定行或某些行，则必须用行寻址（line addressing）。 

在sed编辑器中有两种形式的行寻址： 
- 以数字形式表示行区间 
- 用文本模式来过滤出行 

### 1、数字方式的行寻址

```bash
cat test1.txt
sed 's/dog/cat/' test1.txt
sed '2s/dog/cat/' test1.txt
sed '2,4s/dog/cat/' test1.txt
sed '2,$/dog/cat/' test1.txt
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/sed2.1.jpg"/>


### 2、使用文本模式过滤器

`/pattern/command`必须用正斜线将要指定的pattern封起来。

```bash
#方法一
cat /etc/passwd|tail | sed '/apache/s/nologin/modification/' 
#方法二
cat /etc/passwd|tail | sed '/apache/s$nologin$modification$'
sed '/root/s/bash/modification/' /etc/passwd
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/sed2.2.jpg"/>

### 3、命令组合

如果需要在单行上执行多条命令，可以用花括号将多条命令组合在一起。sed编辑器会处理 地址行处列出的每条命令。 

```bash
sed '2{
> s/fox/elephant/
> s/dog/cat/
> }' test1.txt

sed '3,${
> s/fox/elephant/
> s/dog/cat/
> }' test1.txt
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/sed2.3.jpg"/>

### 3、删除行
- 指定地址

```bash
#不加入寻址模式，流中的所有文本行都会被删除
sed 'd' test2.txt
#加入寻址模式
sed '3d' test2.txt
sed '3,5d' test2.txt
sed '3,$d' test2.txt
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/sed3.1.JPG"/>

- 模式匹配

```bash
sed '/line 2/d' test2.txt
#只要符合匹配模式的行都会被删除，所以下面的命令会删除2行
sed '/line 1/d' test2.txt
#指定的第一个模式 会“打开”行删除功能，第二个模式会“关闭”行删除功能
sed '/2/,/4/d' test2.txt
#因为有两行都匹配到1了，而第二个匹配到的1匹配不到“关闭的模式”，所以后面的内容统统删掉了
sed '/1/,/3/d' test2.txt
sed '/1/,/6/d' test2.txt
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/sed3.2.JPG"/>

## 插入和附加文本

- 插入（insert）命令（i）会在指定行前增加一个新行
- 附加（append）命令（a）会在指定行后增加一个新行


<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/sed4.2.JPG"/>

要向数据流行内部插入或附加数据，必须用寻址来告诉sed编辑器你想让数据出现在什么位置。可以在用这些命令时只指定一个行地址。可以匹配一个数字行号或文本模式，但不能用地址区间。

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/sed4.1.JPG"/>


## 修改行
修改（change）命令允许修改数据流中整行文本的内容。

```bash
#将第一句修改为I like teacher Li.
#方法一：行定位需要修改的行
sed '1c\I like teacher Li.' test2.txt
#方法二：模式匹配定位需要修改的行
sed '/line 1/c\I like teacher Li.' test2.txt
#将1至6行都修改为I like teacher Li.

```

♥因为1至6行都跟teacher Li无关，所以要强行修改一下 ♥

```
sed '1,6c\I like teacher Li.' test2.txt
```


<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/sed5.JPG"/>

## 转换命令

转换（transform）命令（y）是唯一可以处理单个字符的sed编辑器命令。

[address]y/inchars/outchars/ 

转换命令会对inchars和outchars值进行一对一的映射。inchars中的第一个字符会被转 换为outchars中的第一个字符，第二个字符会被转换成outchars中的第二个字符。这个映射过 程会一直持续到处理完指定字符。如果inchars和outchars的长度不同，则sed编辑器会产生一条错误消息。

```bash
#会发现转换命令是全局的
sed 'y/123/456' test2.txt
#匹配行进行修改
sed '1y/1/6' test2.txt
sed '6y/1/6' test2.txt
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/sed6.JPG"/>

## 回顾打印

- p命令用来打印文本行 
- 等号（=）命令用来打印行号
- l（小写的L）命令用来列出行

### 1、打印行

```bash
#模式匹配到的行打印出来
sed -n '/li/p' test2.txt
#行数匹配到的行打印出来
sed -n '7,8p' tets2.txt
#打印出修改之前的行和修改之后的行
sed -n '/qinqin/{
> p
> s/meimei/baobei/p
> }' test2.txt
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/sed7.1.JPG"/>

### 2、打印行号

等号命令会打印行在数据流中的当前行号。行号由数据流中的换行符决定。每次数据流中出 现一个换行符，sed编辑器会认为一行文本结束了。 

在数据流中查找特定文本模式的行，并将其与行号一起打印出来。

```bash
sed -n '/Li/{
> =
> p
}' test2.txt
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/sed7.2.JPG"/>


## 使用sed处理文件

### 1、写入文件

w命令用来向文件写入行。该命令的格式如下：

[address]w filename 

```bash
#支持行匹配和模式匹配
sed '7,8w li.txt' test2.txt
sed '/Li/w Li.txt' test2.txt
sed -n '/qinqin/w qinqin.txt' test2.txt
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/sed8.1.JPG"/>


### 2、从文件读取数据

已经了解了如何在sed命令行上向数据流中插入或附加文本。读取（read）命令（r）允许将一个独立文件中的数据插入到数据流中。 

读取命令的格式如下：

[address]r filename 

支持行寻址和模式匹配

```bash
sed '4r yinyu.txt' friends.txt
sed '4r yinyu.txt' friends.txt
sed '/yuqi/r yinyu.txt' friends.txt
```
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/sed8.2.JPG"/>

```bash
sed '/friends/{
r friends.txt
}' beautiful-people.txt

sed '/friends/{
r friends.txt
d}' beautiful-people.txt
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/sed8.3.JPG"/>
