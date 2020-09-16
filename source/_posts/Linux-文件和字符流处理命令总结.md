---
title: Linux 文件和字符流处理命令总结
date: 2020-09-15 13:46:27
tags:	
	- sort
	- cut
	- wc
	- uniq
	- sed
	- awk
---

> 和计算机打交道，总避免不了和文件操作，简单的文件操作比如 排序、查找、去重等，涉及到这些操作总不能打开文件一行一行的自己手动去操作吧，但是又不想为了处理这些文件写一些脚本代码，那么此时，就可以借助 Linux 内建的 一些命令来处理啦。该篇文章也是一个简单记录总结，供后续自己查看

<!-- more -->

### sort  cut  uniq  wc  sed  awk 六条命令简单记录

### sort

+ sort
  + sort 命令对 File 参数指定的文件中的行排序，并将结果写到标准输出。如果 File 参数指定多个文件，那么 sort 命令将这些文件连接起来，并当作一个文件进行排序。
+ sort 语法
  + sort [-fbMnrtuk] [file or stdin]
  + 选项与参数：
    -f  ：忽略大小写的差异，例如 A 与 a 视为编码相同；
    -b  ：忽略最前面的空格符部分；
    -M  ：以月份的名字来排序，例如 JAN, DEC 等等的排序方法；
    -n  ：使用『纯数字』进行排序(默认是以文字型态来排序的)；
    -r  ：反向排序；
    -u  ：就是 uniq ，相同的数据中，仅出现一行代表；
    -t  ：分隔符，默认是用 [tab] 键来分隔；
    -k  ：以那个区间 (field) 来进行排序的意思
+ 例子
  + cat /etc/passwd | sort
    + sort 是默认以第一个数据来排序，而且默认是以字符串形式来排序,所以由字母 a 开始升序排序。
  + cat /etc/passwd | sort -t ':' -k 3
    + 以 : 来分隔的，以第三栏来排序
    + 默认是以字符串来排序的，如果想要使用数字排序： cat /etc/passwd | sort -t ':' -k 3n
    + cat /etc/passwd | sort -t ':' -k 3nr    r（倒序）
  + cat /etc/passwd |  sort -t':' -k 6.2,6.4 -k 1r 
    + 以第六个域的第2个字符到第4个字符进行正向排序，再基于第一个域进行反向排序
  + cat /etc/passwd |  sort -t':' -k 7 -u
    + 查看/etc/passwd有多少个shell:对/etc/passwd的第七个域进行排序，然后去重

### uniq

+ uniq
  + uniq命令可以去除排序过的文件中的重复行，因此uniq经常和sort合用。也就是说，为了使uniq起作用，所有的重复行必须是相邻的。
+ uniq 语法
  + 选项与参数：
    -i   ：忽略大小写字符的不同；
    -c  ：进行计数
    -u  ：只显示唯一的行
+ 假设testfile的内容如下
  + hello
    world
    friend
    hello
    world
    hello
+ 直接删除未经排序的文件，将会发现没有任何行被删除
  + uniq testfile  输出如下，没有任何改变
  + hello
    world
    friend
    hello
    world
    hello
+ 排序文件，默认是去重
  + cat testfile | sort |uniq
  + 输出
  + friend
    hello
    world
+ 排序之后删除了重复行，同时在行首位置输出该行重复的次数
  + sort testfile | uniq -c
  + 输出
  + 1 friend
    3 hello
    2 world
+ 仅显示存在重复的行，并在行首显示该行重复的次数
  + sort testfile | uniq -dc
  + 输出
  + 3 hello
    2 world
+ 仅显示不重复的行
  + sort testfile | uniq -u
  + 输出 friend  



### cut

+ cut
  + cut命令可以从一个文本文件或者文本流中提取文本列。
+ cut语法
  + 选项与参数：
    -d  ：后面接分隔字符。与 -f 一起使用；
    -f  ：依据 -d 的分隔字符将一段信息分割成为数段，用 -f 取出第几段的意思；
    -c  ：以字符 (characters) 的单位取出固定字符区间；
+ 将 PATH 变量取出，我要找出第五个路径。
  + echo $PATH | cut -d ':' -f 5
+ 将 PATH 变量取出，我要找出第三和第五个路径。
  + echo $PATH | cut -d ':' -f 3,5
+ 将 PATH 变量取出，我要找出第三到最后一个路径。
  + echo $PATH | cut -d ':' -f 3-
+ 将 PATH 变量取出，我要找出第一到第三个路径。
  + echo $PATH | cut -d ':' -f 1-3
+ 将 PATH 变量取出，我要找出第一到第三，还有第五个路径。
  + echo $PATH | cut -d ':' -f 1-3,5
+ 实用例子:只显示/etc/passwd的用户和shell
  + cat /etc/passwd | cut -d ':' -f 1,7

### wc

+ wc
  + 统计文件里面有多少单词，多少行，多少字符。
+ 语法
  + 选项与参数：
    -l  ：仅列出行；
    -w  ：仅列出多少字(英文单字)；
    -m  ：多少字符；

### sed

+ sed
  + sed 是一种在线编辑器，它一次处理一行内容。处理时，把当前处理的行存储在临时缓冲区中，称为“模式空间”（pattern space），接着用sed命令处理缓冲区中的内容，处理完成后，把缓冲区的内容送往屏幕。接着处理下一行，这样不断重复，直到文件末尾。文件内容并没有 改变，除非你使用重定向存储输出。Sed主要用来自动编辑一个或多个文件；简化对文件的反复操作；编写转换程序等。以下介绍的是Gnu版本的Sed 3.02。
+ 命令
  + 调用sed命令有两种形式：
  + sed [options] 'command' file(s)
  + sed [options] -f scriptfile file(s)
+ 删除：d命令
  + sed '2d' example-----删除example文件的第二行。
  + sed '2,$d' example-----删除example文件的第二行到末尾所有行。
  + sed '$d' example-----删除example文件的最后一行。
  + sed '/test/'d example-----删除example文件所有包含test的行。
+ 替换：s命令
  + sed 's/test/mytest/g' example-----在整行范围内把test替换为mytest。如果没有g标记，则只有每行第一个匹配的test被替换成mytest。
  + sed -n 's/^test/mytest/p' example-----(-n)选项和p标志一起使用表示只打印那些发生替换的行。也就是说，如果某一行开头的test被替换成mytest，就打印它。
  + sed 's/^192.168.0.1/&localhost/' example-----&符号表示替换换字符串中被找到的部份。所有以192.168.0.1开头的行都会被替换成它自已加 localhost，变成192.168.0.1localhost。
  + sed -n 's/\(love\)able/\1rs/p' example-----love被标记为1，所有loveable会被替换成lovers，而且替换的行会被打印出来。
  + sed 's#10#100#g' example-----不论什么字符，紧跟着s命令的都被认为是新的分隔符，所以，“#”在这里是分隔符，代替了默认的“/”分隔符。表示把所有10替换成100。
+ 选定行的范围：逗号
  + sed -n '/test/,/check/p' example-----所有在模板test和check所确定的范围内的行都被打印。
  + sed -n '5,/^test/p' example-----打印从第五行开始到第一个包含以test开始的行之间的所有行。
  + sed '/test/,/check/s/$/sed test/' example-----对于模板test和west之间的行，每行的末尾用字符串sed test替换。
+ 多点编辑：e命令
  + sed -e '1,5d' -e 's/test/check/' example-----(-e)选项允许在同一行里执行多条命令。如例子所示，第一条命令删除1至5行，第二条命令用check替换test。命令的执 行顺序对结果有影响。如果两个命令都是替换命令，那么第一个替换命令将影响第二个替换命令的结果。
  + sed --expression='s/test/check/' --expression='/love/d' example-----一个比-e更好的命令是--expression。它能给sed表达式赋值。
+ 从文件读入：r命令
  + sed '/test/r file' example-----file里的内容被读进来，显示在与test匹配的行后面，如果匹配多行，则file的内容将显示在所有匹配行的下面。
+ 写入文件：w命令
  + sed -n '/test/w file' example-----在example中所有包含test的行都被写入file里。
+ 追加命令：a命令
  + sed '/^test/a\\--->this is a example' example    '----->this is a example'被追加到以test开头的行后面，sed要求命令a后面有一个反斜杠。
+ 插入：i命令
  + sed '/test/i\ new line' example  如果test被匹配，则把反斜杠后面的文本插入到匹配行的前面。
+ 下一个：n命令
  + sed '/test/{ n; s/aa/bb/; }' example-----如果test被匹配，则移动到匹配行的下一行，替换这一行的aa，变为bb，并打印该行，然后继续。
+ 变形：y命令
  + sed '1,10y/abcde/ABCDE/' example-----把1--10行内所有abcde转变为大写，注意，正则表达式元字符不能使用这个命令。
+ 退出：q命令
  + sed '10q' example-----打印完第10行后，退出sed。
+ 保持和获取：h命令和G命令
  + sed -e '/test/h' -e '$G example-----在sed处理文件的时候，每一行都被保存在一个叫模式空间的临时缓冲区中，除非行被删除或者输出被取消，否则所有被处理的行都将 打印在屏幕上。接着模式空间被清空，并存入新的一行等待处理。在这个例子里，匹配test的行被找到后，将存入模式空间，h命令将其复制并存入一个称为保 持缓存区的特殊缓冲区内。第二条语句的意思是，当到达最后一行后，G命令取出保持缓冲区的行，然后把它放回模式空间中，且追加到现在已经存在于模式空间中 的行的末尾。在这个例子中就是追加到最后一行。简单来说，任何包含test的行都被复制并追加到该文件的末尾
+ 保持和互换：h命令和x命令
  + sed -e '/test/h' -e '/check/x' example -----互换模式空间和保持缓冲区的内容。也就是把包含test与check的行互换。
+ 脚本
  + Sed脚本是一个sed的命令清单，启动Sed时以-f选项引导脚本文件名。Sed对于脚本中输入的命令非常挑剔，在命令的末尾不能有任何空白或文本，如果在一行中有多个命令，要用分号分隔。以#开头的行为注释行，且不能跨行。

### awk

[说明文档连接](http://www.gnu.org/software/gawk/manual/gawk.html)

[软一峰扫盲](https://www.ruanyifeng.com/blog/2018/11/awk.html)

+ awk

  + awk是一个强大的文本分析工具，相对于grep的查找，sed的编辑，awk在其对数据分析并生成报告时，显得尤为强大。简单来说awk就是把文件逐行的读入，以空格为默认分隔符将每行切片，切开的部分再进行各种分析处理。
  + awk有3个不同版本: awk、nawk和gawk，未作特别说明，一般指gawk，gawk 是 AWK 的 GNU 版本。
  + awk其名称得自于它的创始人 Alfred Aho 、Peter Weinberger 和 Brian Kernighan 姓氏的首个字母。实际上 AWK 的确拥有自己的语言： AWK 程序设计语言 ， 三位创建者已将它正式定义为“样式扫描和处理语言”。它允许您创建简短的程序，这些程序读取输入文件、为数据排序、处理数据、对输入执行计算以及生成报表，还有无数其他的功能。

+ 使用方法

  + awk '{pattern + action}' {filenames}
  + 尽管操作可能会很复杂，但语法总是这样，其中 pattern 表示 AWK 在数据中查找的内容，而 action 是在找到匹配内容时所执行的一系列命令。花括号（{}）不需要在程序中始终出现，但它们用于根据特定的模式对一系列指令进行分组。 pattern就是要表示的正则表达式，用斜杠括起来。
  + awk语言的最基本功能是在文件或者字符串中基于指定规则浏览和抽取信息，awk抽取信息后，才能进行其他文本操作。完整的awk脚本通常用来格式化文本文件中的信息。
  + 通常，awk是以文件的一行为处理单位的。awk每接收文件的一行，然后执行相应的命令，来处理文本。
  + 调用awk有三种方式
  + 1.命令行方式
    + awk [-F  field-separator]  'commands'  input-file(s)
    + 其中，commands 是真正awk命令，[-F域分隔符]是可选的。 input-file(s) 是待处理的文件。在awk中，文件的每一行中，由域分隔符分开的每一项称为一个域。通常，在不指名-F域分隔符的情况下，默认的域分隔符是空格。
  + 2.shell脚本方式
    + 将所有的awk命令插入一个文件，并使awk程序可执行，然后awk命令解释器作为脚本的首行，一遍通过键入脚本名称来调用。相当于shell脚本首行的：#!/bin/sh可以换成：#!/bin/awk
  + 3.将所有的awk命令插入一个单独文件，然后调用：
    + awk -f awk-script-file input-file(s)
      其中，-f选项加载awk-script-file中的awk脚本，input-file(s)跟上面的是一样的。
  + 入门实例
    + 假设last -n 5 的输出如下
    + root     pts/1   192.168.1.100  Tue Feb 10 11:21   still logged in
      root     pts/1   192.168.1.100  Tue Feb 10 00:46 - 02:28  (01:41)
      root     pts/1   192.168.1.100  Mon Feb  9 11:41 - 18:30  (06:48)
      dmtsai   pts/1   192.168.1.100  Mon Feb  9 11:41 - 11:41  (00:00)
      root     tty1                   Fri Sep  5 14:09 - 14:10  (00:01)
  + last -n 5 | awk  '{print $1}'
    + 输出
    + root
      root
      root
      dmtsai
      root
    + awk工作流程是这样的：读入有'\n'换行符分割的一条记录，然后将记录按指定的域分隔符划分域，填充域，\$0则表示所有域,​\$1表示第一个域,\$n表示第n个域。默认域分隔符是"空白键" 或 "[tab]键",所以​\$1表示登录用户，​\$3表示登录用户ip,以此类推。
  + 如果只是显示/etc/passwd的账户
    + cat /etc/passwd |awk  -F ':'  '{print $1}'
    + 这种是awk+action的示例，每行都会执行action{print $1}。
    + -F指定域分隔符为':'。
  + 如果只是显示/etc/passwd的账户和账户对应的shell,而账户与shell之间以tab键分割
    + cat /etc/passwd |awk  -F ':'  '{print \$1"\t"$7}'
  + 如果只是显示/etc/passwd的账户和账户对应的shell,而账户与shell之间以逗号分割,而且在所有行添加列名name,shell,在最后一行添加"blue,/bin/nosh"。
    + cat /etc/passwd |awk  -F ':'  'BEGIN {print "name,shell"}  {print \$1","$7} END {print "blue,/bin/nosh"}'
    + awk工作流程是这样的：先执行BEGING，然后读取文件，读入有/n换行符分割的一条记录，然后将记录按指定的域分隔符划分域，填充域，\$0则表示所有域,\$1表示第一个域,$n表示第n个域,随后开始执行模式所对应的动作action。接着开始读入第二条记录······直到所有的记录都读完，最后执行END操作。
  + 搜索/etc/passwd有root关键字的所有行
    + awk -F: '/root/' /etc/passwd
    + 这种是pattern的使用示例，匹配了pattern(这里是root)的行才会执行action(没有指定action，默认输出每行的内容)。
    + 搜索支持正则，例如找root开头的: awk -F: '/^root/' /etc/passwd
  + 搜索/etc/passwd有root关键字的所有行，并显示对应的shell
    + awk -F: '/root/{print $7}' /etc/passwd   
    + 这里指定了action{print $7}
  + awk内置变量
    + awk有许多内置变量用来设置环境信息，这些变量可以被改变，下面给出了最常用的一些变量。
    + ARGC               命令行参数个数
      ARGV               命令行参数排列
      ENVIRON            支持队列中系统环境变量的使用
      FILENAME           awk浏览的文件名
      FNR                浏览文件的记录数
      FS                 设置输入域分隔符，等价于命令行 -F选项
      NF                 浏览记录的域的个数
      NR                 已读的记录数
      OFS                输出域分隔符
      ORS                输出记录分隔符
      RS                 控制记录分隔符
  + 统计/etc/passwd:文件名，每行的行号，每行的列数，对应的完整行内容:
    + awk  -F ':'  '{print "filename:" FILENAME ",linenumber:" NR ",columns:" NF ",linecontent:"$0}' /etc/passwd
  + 使用printf替代print,可以让代码更加简洁，易读
    + awk  -F ':'  '{printf("filename:%s,linenumber:%s,columns:%s,linecontent:%s\n",FILENAME,NR,NF,$0)}' /etc/passwd

  #### awk编程

  + 变量和赋值

  + 除了awk的内置变量，awk还可以自定义变量。

  + 下面统计/etc/passwd的账户人数

    + awk '{count++;print $0;} END{print "user count is ", count}' /etc/passwd
    + count是自定义变量。之前的action{}里都是只有一个print,其实print只是一个语句，而action{}可以有多个语句，以;号隔开。

  + 这里没有初始化count，虽然默认是0，但是妥当的做法还是初始化为0:

    + awk 'BEGIN {count=0;print "[start]user count is ", count} {count=count+1;print $0;} END{print "[end]user count is ", count}' /etc/passwd

  + 统计某个文件夹下的文件占用的字节数

    + ls -l |awk 'BEGIN {size=0;} {size=size+$5;} END{print "[end]size is ", size}'

  + 如果以M为单位显示:

    + ls -l |awk 'BEGIN {size=0;} {size=size+$5;} END{print "[end]size is ", size/1024/1024,"M"}'
    + 注意，统计不包括文件夹的子目录。

  + 条件语句

    + awk中的条件语句是从C语言中借鉴来的，见如下声明方式：

    + if (expression) {
          statement;
          statement;
          ... ...
      }

      if (expression) {
          statement;
      } else {
          statement2;
      }

      if (expression) {
          statement1;
      } else if (expression1) {
          statement2;
      } else {
          statement3;
      }

  + 统计某个文件夹下的文件占用的字节数,过滤4096大小的文件(一般都是文件夹):

    + ls -l |awk 'BEGIN {size=0;print "[start]size is ", size} {if($5!=4096){size=size+$5;}} END{print "[end]size is ", size/1024/1024,"M"}' 

  + 循环语句

    + awk中的循环语句同样借鉴于C语言，支持while、do/while、for、break、continue，这些关键字的语义和C语言中的语义完全相同。

  + 数组

    + 因为awk中数组的下标可以是数字和字母，数组的下标通常被称为关键字(key)。值和关键字都存储在内部的一张针对key/value应用hash的表格里。由于hash不是顺序存储，因此在显示数组内容时会发现，它们并不是按照你预料的顺序显示出来的。数组和变量一样，都是在使用时自动创建的，awk也同样会自动判断其存储的是数字还是字符串。一般而言，awk中的数组用来从记录中收集信息，可以用于计算总和、统计单词以及跟踪模板被匹配的次数等等。
    + 显示/etc/passwd的账户
    + awk -F ':' 'BEGIN {count=0;} {name[count] = $1;count++;}; END{for (i = 0; i < NR; i++) print i, name[i]}' /etc/passwd

  > awk 博大精深，这是简单介绍



