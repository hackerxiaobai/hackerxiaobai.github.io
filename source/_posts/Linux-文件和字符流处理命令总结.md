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

### sort  cut  uniq   wc 四条命令简单记录

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

