---
title: 初步认识sed和gwak
date: 2020-03-07 14:31:16
tags: Linux
---
想在shell脚本中处理任何类型的数据，就要熟悉Linux中的sed和gawk工具。Linux世界中广泛使用的两个命令行编辑器：sed和gawk
<!--more-->

# sed编辑器
>sed编辑器被称作流编辑器（stream editor），和普通的交互式文本编辑器恰好相反。在交互式文本编辑器中（比如vim），你可以用键盘命令来交互式地插入、删除或替换数据中的文本。流编辑器则会在编辑器处理数据之前基于预先提供的一组规则来编辑数据流。

## 1.再命令行定义编辑器命令
- 可以直接讲数据通过管道得形式传给sed编辑器。

```bash
echo "This is a test"|sed 's/test/final test'
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/基础sed1.JPG"/>

sed编辑器并不会修改文本文件的数据。它只会将修改后的数据发送到 STDOUT。如果你查看原来的文本文件，它仍然保留着原始数据。 

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/基础sed2.JPG"/>


## 2、在命令行使用多个编辑器命令
- 要在sed命令行上执行多个命令时，只要用-e选项就可以了。
```bash
#多条命令间用分号隔开，分号和命令间不能有空格
sed -e 's/brown/green/;s/dog/cat/' test1.txt
#也可以用此提示符来分隔命令
sed -e '
> s/brown/green/
> s/fox/algea/
> s/dog/cat/' test1.txt
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/基础sed3.JPG"/>

## 3、从文件中读取编辑器命令

- 如果有大量要处理的sed命令，那么将它们放进一个单独的文件中通常会更方便一些。 可以在sed命令中用-f选项来指定文件。

```bash
sed -f script1.sed test1.txt
```
用.sed作为sed脚本文件的扩展名，以避免与bash shell脚本文件搞混。

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/基础sed4.JPG"/>


# gawk程序

## 1、gawk命令格式

```bash
gawk options program file
```

## 2、从命令行读取程序脚本

```bash
gawk 'print "I like teacher Li"'
```

- 由于程序脚本被设为显示一行固定的文 本字符串，因此不管你在数据流中输入什么文本，都会得到同样的文本输出。
- bash shell提供了一个组合键来生成 EOF（End-of-File）字符。`Ctrl+D`组合键会在bash中产生一个EOF字符。这个组合键能够终止该gawk 程序并返回到命令行界面提示符下。
  
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/基础gawk1.JPG"/>

## 3、使用数据字段变量

- $0代表整个文本行
- $1代表文本行中的第1个数据字段 
- $2代表文本行中的第2个数据字段 
- $n代表文本行中的第n个数据字段

```bash
gawk '{print $1}' test1.txt
gawk '{print $2}' test1.txt
```

gawk中默认的字段分隔符是任意的空白字符（例如空格或制表符）。

如果你要读取采用了其他字段分隔符的文件，可以用-F选项指定。

/etc/passwd文件用冒号来分隔数字字段，因而如果要划分开每个数据元素，则必须在gawk选项中将冒号指定为字段分隔符。

```
cat /etc/passwd
gawk -F: '{print $1}' /etc/passwd
```
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/基础gawk2.JPG"/>

## 4、在程序脚本中使用多个命令

```
#在命令行上的程序脚本中使用多条命令
echo "I like my teacher Li"|gawk '{$4="qinqin";$5="meimei";print $0}'

#也可以用次提示符一次一行地输入程序脚本命令
gawk '{
> $4="qinqin"
> $5="meimei"
> print $0}'
I like my teacher Li
```

因为没有在命令行中指定文件名，gawk程序会 从STDIN中获得数据。当运行这个程序的时候，它会等着读取来自STDIN的文本。要退出程序， 只需按下Ctrl+D组合键来表明数据结束。 

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/基础gawk3.JPG"/>

## 5、从文本中读取程序

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/基础gawk4.1.JPG"/>

`gawk`程序在引用变量值时并未像shell脚本一样使用美元符。 

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/基础gawk4.2.JPG"/>

## 6、在处理数据前运行脚本
有时可能需要在处理数据前运行脚本，比如为报告创建标题。`BEGIN`就可以做到。`BEGIN`强制`gawk`在读取数据前执行BEGIN关键字后指定的程序脚本。 

- 这次print命令会在读取数据前显示文本。但在它显示了文本后，它会快速退出，不等待任 何数据。


```bash
gawk 'BEGIN {print "the content of gawk.txt"}'
```

- 如果想使用正常的程序脚本中处理数据，必须用另一个脚本区域来定义程序。 

```bash
gawk 'BEGIN {print "the content of gawk.txt"}
> {print $0}' gawk.txt
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/基础gawk5.JPG"/>

## 7、在处理数据后运行脚本

- 与BEGIN关键字类似，END关键字允许你指定一个程序脚本，gawk会在读完数据后执行它。

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/基础gawk6.1.JPG"/>

- 也可以将一组命令组合起来形成一个小脚本文件，同样使用`-f`选项执行脚本中的命令。

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/基础gawk6.JPG"/>
