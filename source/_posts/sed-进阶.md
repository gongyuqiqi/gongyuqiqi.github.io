---
title: sed 进阶
date: 2020-03-09 09:43:03
tags: Linux
---

多行命令、保持空间、排除命令、改变流、模式替代、在脚本中使用sed、创建sed实用程序

<!--more-->

# sed进阶

## 一、多行命令

**sed编辑器包含了三个可用来处理多行文本的特殊命令。**
- N：将数据流中的下一行加进来创建一个多行组（multiline group）来处理 
- D：删除多行组中的一行 
- P：打印多行组中的一行
### （一）next命令
#### 1、单行的next命令

```bash
#sed命令默认匹配所有符合条件的行
sed '/^\s*$/d' data1.txt
#模式匹配到含有header的那一行，n为跳到下一行，d为删掉n跳到的那行
sed '/header/{n;d}' data.txt
sed '/data/{n;d}' data.txt
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/1.1.JPG"/>


#### 2、合并文本行 

单行next命令会将数据流中的下一文本 行移动到sed编辑器的工作空间（称为模式空间）。**多行版本的next命令（用大写N）会将下一文本行添加到模式空间中已有的文本后。**
 
这样的作用是**将数据流中的两个文本行合并到同一个模式空间中**。文本行仍然用换行符分隔，但sed编辑器现在会将两行文本当成一行来处理。

```bash
#sed编辑器脚本查找含有单词first的那行文本。找到该行后，它会用N命令将下一行合并到那 行，然后用替换命令s将换行符替换成空格
sed '/first/{N; s/\n/ /}' data2.txt
#一下两行代码效果一样。因为data3.txt中的System和Administrator间为换行符，所以使用N也没有用
sed 's/System Administrator/Desktop User/' data3.txt 
sed 'N; s/System Administrator/Desktop User/' data3.txt 
#替换命令在System和Administrator之间用了通配符模式（.）来匹配空格和换行符,当它匹配了换行符时，它就从字符串中删掉了换行符，导致两行合并成一行
sed 'N; s/System.Administrator/Desktop User/' data3.txt
#可以用一下方式避免两行匹配
sed 'N
> s/System\nAdministrator/Desktop\nUser/
> s/System Administrator/Desktop User/
> ' data3.txt
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/1.2.1.JPG"/>


```bash
#这个脚本总是在执行sed编辑器命令前将下一行文本读入到模式空间。当它到了最后一行文本时，就没有下一行可读了，所以N命令会叫sed编辑器停止。如果要匹配的文本正好在数据流的最后一行上，命令就不会发现要匹配的数据。 
sed 'N
s/System\nAdministrator/Desktop\nUser/
s/System Administrator/Desktop User/
' data4.txt
#由于System Administrator文本出现在了数据流中的后一行，N命令会错过它，因为没有其他行可读入到模式空间跟这行合并。将单行命令放到N命令前面，并将多行命令放到N命令后面就可以得到理想的结果。
sed '
> s/System Administrator/Desktop User/
> N
> s/System\nAdministrator/Desktop User/
> ' data4.txt
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/1.2.2.JPG"/>

### （二）多行删除命令

```bash
#很奇怪，我的centos7环境下 sed '/^$/d' data5.txt似乎失灵了。
#删除所有空白行
sed '/^\s*$/d' data5.txt
#sed编辑器脚本会查找空白行，然后用N命令来将下一文本行添加到模式空间。如果新的模式空间内容含有单词header，则D命令会删除模式空间中的第一行。
sed '/^\s*$/{N; /header/D}' data5.txt
sed '/^\s*$/{N; /last/D}' data5.txt
#删除命令会在不同的行中查找单词System和Administrator，然后在模式空间中将两行都删掉。 
sed 'N; /System\nAdministrator/d' data4.txt
#sed编辑器提供了多行删除命令D，它只删除模式空间中的第一行。该命令会删除到换行符（含换行符）为止的所有字符。
sed 'N; /System\nAdministrator/D' data4.txt
```
如果不结合使用N命令和D命令， 就不可能在不删除其他空白行的情况下只删除第一个空白行。 

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/1.2D.JPG"/>

### （三）多行打印命令

当多行匹配出现时，P命令只会打印模式空间中的第一行。
`-n`选项禁止标准输出，值输出P的内容。

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/1.3.JPG"/>

## 二、保存空间

- 模式空间（pattern space）是一块活跃的缓冲区，在sed编辑器执行命令时它会保存待检查的文本。但它并不是sed编辑器保存文本的唯一空间。
- sed编辑器有另一块称作保持空间（hold space）的缓冲区域。在处理模式空间中的某些行时，可以用保持空间来临时保存一些行。
- 
**`p`命令是打印当前模式空间的内容**

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/2.1.JPG"/>

```bash
sed -n '/first/{h;p;n;p;g;p}' data2.txt
```
- sed脚本在地址中用正则表达式来过滤出含有单词first的行
- 当含有单词first的行出现时，h命令将该行放到保持空间
- p命令打印模式空间也就是第一个数据行的内容
- n命令提取数据流中的下一行（This is the second data line），并将它放到模式空间
- p命令打印模式空间的内容，现在是第二个数据行
- g命令将保持空间的内容（This is the first data line）放回模式空间，替换当前文本
- p命令打印模式空间的当前内容，现在变回第一个数据行了

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/2.2.JPG"/>

## 三、排除命令

感叹号命令（!）用来排除（negate）命令，也就是让原本会起作用的命令不起作用。

```bash
#打印模式匹配的行
sed -n '/header/p' data2.txt
#打印除了模式匹配的行
sed -n '/header/!p' data2.txt
#N命令会在最后一行结束，因为后面没有更多的行与最后一行合并了
sed 'N
s/System\nAdministrator/Desktop\nUser/
s/System Administrator/Desktop User/
' data4.txt
#N命令可以作用于除了最后一行的所有行
sed '$!N
s/System\nAdministrator/Desktop\nUser/
s/System Administrator/Desktop User/
' data4.txt
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/3.1.JPG"/>

```bash
cat data2.txt
#倒序输出文本内容
tac data2.txt
#也可以用sed命令实现倒序输出，既打印最后一次的模式空间的内容
sed -n '{1!G;h;$p}' data2.txt
#可以查看每次模式空间的内容
sed -n '{1!G;h;p}' data2.txt
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/3.2.JPG"/>

## 四、改变流

### （一）、分支

将模式匹配的行定义一个标签，满足模式匹配的行执行标签后的命令，不满足模式匹配的行执行非标签后的命令

```bash
#此处没有定义标签，既满足模式匹配的行不进行任何操作，其他行执行给定的替换操作
sed '{2,3b;s/This is/Is this/;s/line./test?/}' data2.txt 
#以下三种方法效果一样
#①
sed '{/header/b lable; s/This is/Is this/
:lable                     
s/This is/THIS IS/}' data2.txt
#②
sed '{/header/b lable;s/This is/Is this/;s/line./test?/;:lable s/This is/THIS IS/}' data2.txt 
#③
sed '/header/b lable;s/This is/Is this/;s/line./test?/;:lable s/This is/THIS IS/' data2.txt 
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/4.1.JPG"/>

上面的例子演示了跳转到sed脚本后面的标签上。也可以跳转到脚本中靠前面的标签上，这样就达到了**循环**的效果。 

```bash
#此方法会形成了一个无穷循环，不停地查找逗号，直到使用Ctrl+C组合键发送一个信号， 手动停止这个脚本
$echo "I, like, my, teacher, Li."|sed -n '{
:lable
s/,//1p
b lable}'
#加上模式匹配，就会在模式匹配是执行替换命令，无法匹配时停止脚本
$echo "I, like, my, teacher, Li."|sed -n '{
:lable
s/,//1p
/,/b lable}'
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/4.2.JPG"/>

### （二）测试

测试命令会根据替换命令的结果跳转到某个标签，而不是根据地址进行跳转

测试命令使用与分支命令相同的格式：

[address]t [label] 

```bash
#第一个替换命令会查找模式文本first。如果匹配了行中的模式，它就会替换文本，而且测 试命令会跳过后面的替换命令。如果第一个替换命令未能匹配模式，第二个替换命令就会被执行
sed '{
s/first/matched/
t
s/This is the/No match on/}' data2.txt
#如果替换命令成功匹配并替换了一个模式，测试命令就会跳转到指定的标签。如果替换命令 未能匹配指定的模式，测试命令就不会跳转。
#有了测试命令，就能结束之前用分支命令形成的无限循环。  
$echo "I, like, my, teacher, Li."|sed -n '{
> :lable
> s/,//p
> t lable}'

```
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/4.3.JPG"/>


## 五、模式替代

### （一）&符号

匹配模式中的一个单 词，那就非常简单。

```bash
echo "The cat sleeps in his hat." | sed 's/cat/"cat"/' 
```
 
但如果你在模式中用通配符（.）来匹配多个单词呢?

```bash
#默认替换匹配的第一个
 echo "The cat sleeps in his hat." | sed 's/.at/".at"/' 
#加上g就可以全局匹配
 echo "The cat sleeps in his hat." | sed 's/.at/".at"/g' 
#但是，只是想对符合.at模式的单词添加引号，而上述方法不能得到理想的结果
```

`&`符号可以用来代表替换命令中的匹配的模式。不管模式匹 配的是什么样的文本，你都可以在替代模式中使用&符号来使用这段文本

```bash
echo "The cat sleeps in his hat." | sed 's/.at/"&"/g' 
```

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/5.1.JPG"/>


### （二）、代替单独的单词
当在替换命令中使用圆括号时，必须用转义字符将它们标示为分组字符而不是普通的圆 括号。这跟转义其他特殊字符正好相反。 

sed编辑器用圆括号来定义替换模式中的子模式。你可以在替代模式中使用特殊字符来引用 每个子模式。替代字符由反斜线和数字组成。数字表明子模式的位置。sed编辑器会给第一个子 模式分配字符\1，给第二个子模式分配字符\2，依此类推。

第一个例子中直接将sleeps替换为空，但其后面的空格依然保留了，用子模式的方法就可以很好的解决。

第二个例子中想要在teacher和Li之间加上`，`号

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/5.2.JPG"/>


## 六、在脚本中使用sed

### （一）、使用包装脚本

命令行的方法倒序文本内容。

脚本的方法倒序文本内容。

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/6.1.JPG"/>


### （二）、重定向sed的输出

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/6.2.JPG"/>


## 七、创建sed实用工具

### （一）、加倍行间距

向文本文件的行间插入空白行的简单sed脚本。

G命令会简单地将保持空 间内容附加到模式空间内容后。当启动sed编辑器时，保持空间只有一个空行。将它附加到已有 行后面，你就在已有行后面创建了一个空白行。

如果不想要这个空白行，可以用排除符号（!）和尾行符号（$）来确 保脚本不会将空白行加到数据流的后一行后面。 

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/7.1.JPG"/>


### （二）、对含有空白行的文件加行间距

直接用G命令会发现：在原来空白行的位置有了三个空白行。这个问题的解决办法是，首先删除数据流中的所有空白行，然后用G命令在所有行后插入新的空白行。要删除已有的空白行，需要将d命令和一个匹配空白行的模式一起使用。 

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/7.2.JPG"/>


### （三）、给文件中的行编号

有些bash shell命令也可以添加行号，但它们会另外加入一些东西（有可能是不需要的间隔）。

可以用等号`=`来显示数据流中行的行号，但阅读上不友好。

另一种办法是：在获得了等号命令的输出之后，你可以通过管道将输出传给另一个sed编辑器脚本，它会使用N命令来合并这两行。还需要用替换命令将换行符更换成空格或制表符。

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/7.3.JPG"/>


### （四）、打印末尾行

循环N命令和D命令，你在向模式空间的文本行块增加新行的同时也删除了旧行。分支命令非常适合这个循环。要结束循环，只要识别出后一行并用q命令退出就可以了。 

第一个sed脚本会首先检查这行是不是数据流中后一行。如果是，退出（quit）命令会停止循环。N命令会将下一行附加到模式空间中当前行之后。如果当前行在第2行后面，3,$D命令会删除模式空间中的第一行。这就会在模式空间中创建出滑动窗口效果。
第二个脚本，如果当前行在第3行后面，4,$D命令会删除模式空间中的第一行。
第三个脚本，如果当前行在第4行后面，5,$D命令会删除模式空间中的第一行。


<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/7.4.JPG"/>

### （五）、删除行

**1、删除连续的空白行**

区间是/./到/^$/。区间的开始地址会匹配任何含有至少一个字符的行。区间的结束地址会 匹配一个空行。在这个区间内的行不会被删除。 

如果开头有空白行的话，开头的空白行也会被删除

**2、删除开头的空白行**

/./,$!d 这个则表示从第一个字符到最后的内容统统不删除，即删除开头的空白行。

**3、删除结尾的空白行**

```bash
sed '{ :start /^\n*$/{$d; N; b start } }' 
```

在正常脚本的花括号里还有花括号。这允许你在整个命令脚本 中将一些命令分组。该命令组会被应用在指定的地址模式上。地址模式能够匹配只含有一个换行 符的行。如果找到了这样的行，而且还是后一行，删除命令会删掉它。如果不是后一行，N 命令会将下一行附加到它后面，分支命令会跳到循环起始位置重新开始。 

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/7.5.JPG"/>


### (六)、删除HTML标签

s/<.*>//g 这种匹配模式会删掉<title>This is the page title</title>等。

s/<[^>]*>//g 这种模式才能解决问题。

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/7.6.JPG"/>
