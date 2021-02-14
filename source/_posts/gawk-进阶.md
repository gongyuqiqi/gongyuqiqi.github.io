---
title: gawk 进阶
date: 2020-03-16 18:18:53
tags: Linux
---

使用变量、处理数组、处理模式

<!--more-->

# gawk进阶

## 一、使用变量

### （一）、内建变量

**字段和记录分隔符变量**

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/g1.1.JPG"/>

print命令会自动将OFS变量的值放置在输出中的每个字段间。通过设置OFS变量，可以在输 出中使用任意字符串来分隔字段。

FIELDWIDTHS变量允许你不依靠字段分隔符来读取记录。在一些应用程序中，数据并没有使 用字段分隔符，而是被放置在了记录中的特定列。这种情况下，必须设定FIELDWIDTHS变量来 匹配数据在记录中的位置。

如果你用默认的FS和RS变量值来读取这组数据，gawk就会把每行作为一条单独的记录来读 取，并将记录中的空格当作字段分隔符。要解决这个问题，只需把FS变量设置成换行符。这就表明数据流中的每行都是一个单独的字 段，每行上的所有数据都属于同一个字段。把RS变量设置成空字符串，然后在数据记录间留一个空白行。gawk会 把每个空白行当作一个记录分隔符。 

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/g1.2.JPG"/>


**数据变量**

跟shell变量不同，在脚本中引用gawk变量时，变量名前不加美元符。 

ARGC和ARGV变量允许从shell中 获得命令行参数的总数以及它们的值。但这可能有点麻烦，因为gawk并不会将程序脚本当成命令行参数的一部分。ARGV数组从索引0开始，代表的是命令。

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/g1.21.JPG"/>

ENVIRON变量使用关联数组来提取shell环境变量。关联数组用文本 作为数组的索引值，而不是数值。

ENVIRON["HOME"]变量从shell中提取了HOME环境变量的值。类似地，ENVIRON["PATH"]提 取了PATH环境变量的值

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/g1.22.JPG"/>


NF变量可以让你在不知道具体位置的情况下指定记录中的后一个数据字段。NF变量含有数据文件中后一个数据字段的数字值。可以在它前面加个美元符将其用作字段 变量。 

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/g1.23.JPG"/>


FNR变量含有当前数据文件中已处理过的记录数，NR变量则含有已处理过的记录总数。

FNR变量的值在gawk处理第二个数据文件时被重置了，而NR变量则在处理第二个数据文件时 继续计数。结果就是：如果只使用一个数据文件作为输入，FNR和NR的值是相同的；如果使用多 个数据文件作为输入，FNR的值会在处理每个数据文件时被重置，而NR的值则会继续计数直到处 理完所有的数据文件。 

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/g1.24.JPG"/>

### （二）、自定义变量

**在脚本中给变量赋值**

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/g2.1.JPG"/>

**在命令行上给变量赋值**

前两个例子可以让在不改变脚本代码的情况下改变脚本的行为。第一个例子显示了文件的第二个数据字段，第二个例子显示了第三个数据字段，只要在命令行上设置n变量的值就行。

使用命令行参数来定义变量值会有一个问题。在你设置了变量后，这个值在代码的BEGIN部分不可用。 

可以用-v命令行参数来解决这个问题。它允许你在BEGIN代码之前设定变量。在命令行上，-v命令行参数必须放在脚本代码之前。变量紧挨-v参数，也要放在脚本之前。

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/g2.2.JPG"/>


## 二、处理数组

为了在单个变量中存储多个值，许多编程语言都提供数组。gawk编程语言使用关联数组提供 数组功能。关联数组跟数字数组不同之处在于它的索引值可以是任意文本字符串。你不需要用连续的数字来标识数组中的数据元素。相反，关联数组用各种字符串来引用值。每个索引字符串都必须能 够唯一地标识出赋给它的数据元素。跟Python中的字典类似。

**定义数组变量**

var[index] = element

其中var是变量名，index是关联数组的索引值，element是数据元素值。下面是一些gawk中数组变量的例子。

`'` 和`"`在本例中效果不一样

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/g2.21.JPG"/>


**遍历数组变量**

关联数组变量的问题在于你可能无法知晓索引值是什么。跟使用连续数字作为索引值的数字数组不同，关联数组的索引可以是任何东西。

如果要在gawk中遍历一个关联数组，可以用for语句的一种特殊形式。

注意：(key in f)才能正确执行，key in f会报错。
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/g2.22.JPG"/>

**删除数组变量**

delete array[index]

删除命令会从数组中删除关联索引值和相关的数据元素值。

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/g2.3.JPG"/>

## 三、使用模式

### （一）、正则表达式

gawk程序会用正则表达式对记录中所有的数据字段进行匹配，包括字段分隔符。 

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/g3.1.JPG"/>

### （二）、匹配操作符

匹配操作符（matching operator）允许将正则表达式限定在记录中的特定数据字段。匹配操作符是波浪线（~）。可以指定匹配操作符、数据字段变量以及要匹配的正则表达式。

$2变量代表记录中的第一个数据字段。这个表达式会过滤出第一个字段以文本2开头的所有记录。data同理。

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/g3.2.1JPG.JPG"/>

也可以用!符号来排除正则表达式的匹配。

$1 !~ /expression/

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/g3.2.2JPG.JPG"/>


### （三）、数学表达式

第一个测试显示所有属于root用户组（组ID为0）的系统用户。

对文本数据使用表达式，必须小心。跟正则表达式不同，表达式必须完全匹配。数据必须跟模式严格匹配。 
第二个测试中没有匹配任何记录，因为第一个数据字段的值不在任何记录中。第三个测试用值data11匹配了一条记录。 

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/g3.3.JPG"/>


## 四、结构化命令

### （一）、if语句

gawk编程语言支持标准的if-then-else格式的if语句。你必须为if语句定义一个求值的条件，并将其用圆括号括起来。如果条件求值为TRUE，紧跟在if语句后的语句会执行。如果条 件求值为FALSE，这条语句就会被跳过。

格式： 

**if (condition)**

**statement1** 

也可以将它放在一行上，像这样：

**if (condition) statement1** 

可以在多行使用else子句,也可以在单行上使用else子句，但必须在if语句部分之后使用分号。

**if (condition) statement1; else statement2** 

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/g4.1.1.JPG"/>

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/g4.1.2.JPG"/>

### （二）、while语句

while语句为gawk程序提供了一个基本的循环功能。下面是while语句的格式。 

**while (condition)**

**{**   
    **statements**  
**}**

while循环允许遍历一组数据，并检查迭代的结束条件。


<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/g4.2.1.JPG"/>

第一个例子和第二个例子中的number=0多余了，忽略它。

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/g4.2.2.JPG"/>

### （三）、do-while语句

do-while语句类似于while语句，但会在检查条件语句之前执行命令。下面是do-while语句的格式。

**do**  
**{**   
    **statements**  
**} while (condition)** 

这种格式保证了语句会在条件被求值之前至少执行一次。当需要在求值条件前执行语句时， 这个特性非常方便。 


<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/g4.3.1.JPG"/>

这个脚本会读取每条记录的数据字段并将它们加在一起，直到累加结果达到4。如果第一个数据字段大于4（就像在第二条和第三条记录中看到的那样），则脚本会保证在条件被求值前至少读取第一个数据字段的内容。

### （四）、for语句

for语句是许多编程语言执行循环的常见方法。gawk编程语言支持C风格的for循环。将多个功能合并到一个语句有助于简化循环。 

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/g4.4.JPG"/>


## 五、格式化打印

**格式化打印命令：printf**

如果你需要显示一个字符串变量，可以用格式化指定符%s。如果你需要显示一个整数 值，可以用%d或%i（%d是十进制数的C风格显示方式）。

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/g5.1.JPG"/>

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/g5.1.2.JPG"/>

你需要在printf命令的末尾手动添加换行符来生成新行。没添加的话，printf命令会继续在同一行打印后续输出。

<font color=blue>每个printf的输出都会出现在同一行上</font>。为了终止该行，END部分打印了一个换行符。 

下面这个例子也可以再次体会RS的作用。FS指定输入的分隔符，RS指定输入记录分隔符。即对记录的数据从新定义的分隔符，所以$1为1，$2为2,$3为3,一次类推。

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/g5.1.5.JPG"/>

printf命令在处理浮点值时也非常方便。通过为变量指定一个格式，你可以让输出看起来更统一。可以使用%.1f格式指定符来强制printf命令将浮点值近似到小数点后一位。  

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/g5.1.3.JPG"/>

**可以用printf命令来帮助格式化输出，使得输出信息看起来更美观。**


◉ width：指定了输出字段小宽度的数字值。如果输出短于这个值，printf会将文本右 对齐，并用空格进行填充。如果输出比指定的宽度还要长，则按照实际的长度输出。

◉ prec：这是一个数字值，指定了浮点数中小数点后面位数，或者文本字符串中显示的 大字符数。

◉ -（减号）：指明在向格式化空间中放入数据时采用左对齐而不是右对齐。 


- 首先，让我们将print 命令转换成printf命令，它会产生跟print命令相同的输出。printf命令用%s格式化指定符来作为这两个字符串值的占位符。printf需要手动换行。


<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/g5.1.4.JPG"/>

- 通过添加一个值为16的修饰符，我们强制第一个字符串的输出宽度为16个字符。默认情况下， printf命令使用右对齐来将数据放到格式化空间中。要改成左对齐，只需给修饰符加一个减号 即可

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/g5.1.6.JPG"/>

## 六 内建函数

### （一）、数学函数
gawk中内建的数学函数。

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/g6.1.1.JPG"/>


int() 函数会生成一个值的整数部分，但它并不会四舍五入取近似值。它的做法像R语言中的floor函数。它会生成该值和0之间接近该值的整数。 int()函数在值为5.6时返回5，在值为-5.6时则返回-5。

rand()函数非常适合创建随机数，但这个随机数只在0和1之间（不包括0或1）。要得到更大的数，就需要放大返 回值。 

产生较大整数随机数的常见方法是用rand()函数和int()函数创建一个算法。

x = int(10 * rand()) 

gawk语言对于它能够处理的数值有一个限定区间。如果 超出了这个区间，就会得到一条错误消息。 

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/g6.1.2.JPG"/>

### （二）、字符串函数

怎么感觉有点麻烦，先不看这个了。😏

### （三）、时间函数

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/g6.3.1.JPG"/>

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/g6.3.2.JPG"/>

## 七、自定义函数

### （一）、定义函数

要定义自己的函数，必须用function关键字。函数名必须能够唯一标识函数。

**function name([variables])**   
**{**    
    **statements**  
**}** 

### （二）、使用自定义函数

在定义函数时，它必须出现在所有代码块之前（包括BEGIN代码块）。乍一看可能有点怪异，但它有助于将函数代码与gawk程序的其他部分分开。

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/g7.2.JPG"/>

### （三）、创建函数库

gawk提供了一种途径来将多个函数放到一个库文件中，这样就能在所有的gawk程序中使用了。

- 首先，你需要创建一个存储所有gawk函数的文件。

- 需要使用-f命令行参数来使用它们。很遗憾，不能将-f命令 行参数和内联gawk脚本放到一起使用，不过可以在同一个命令行中使用多个-f参数。

- 因此，要使用库，只要创建一个含有你的gawk程序的文件，然后在命令行上同时指定库文件和程序文件就行了。

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/g7.3.JPG"/>
