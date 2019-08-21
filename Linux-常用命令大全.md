---
title: Linux-常用命令大全
date: 2018-03-04 15:54:56
categories: 
- Linux
tags:
- Linux
---

Linux命令大全

原文链接：[Linux命令大全----常用文件操作命令](http://blog.csdn.net/Evankaka/article/details/49227669)

<!--more-->

## 一、ls

| 命令（注意空格）                           | 解释                                                         |
| ------------------------------------------ | ------------------------------------------------------------ |
| ls -a                                      | 显示全部文件，包含隐藏文件                                   |
| ls –F                                      | 使用这个参数表示在文件的后面多添加表示文件类型的符号，例如*表示可执行，/表示目录，@表示连结文件，这都是因为使用了-F这个参数。 |
| ls -l （这个参数是字母L的小写，不是数字1） | 使用长格式显示文件内容，如果需要察看更详细的文件资料         |



```shell
-rw-r--r--@  1 haozhu  staff      27197 11  9 15:30 JavaWebSocket.zip
drwxr-xr-x@  7 haozhu  staff        224  8 23  2016 Kafka_study_demo-master
-rw-r--r--@  1 haozhu  staff   10465449  1  8 16:13 Kafka_study_demo-master.zip
-rw-r--r--@  1 haozhu  staff     455705 11 20 16:02 SpringMVC-Mybatis-Shiro-redis-0.2-master.zip
```



>　第一个栏位，表示文件的属性。Linux的文件基本上分为三个属性：可读（r），可写（w），可执行（x）。但是这里有十个格子可以添（具体程序实现时，实际上是十个bit位）。第一个小格是特殊表示格，表示目录或连结文件等等，d表示目录，例如drwx------;l表示连结文件，如lrwxrwxrwx;如果是以一横“-”表示，则表示这是文件。其余剩下的格子就以每3格为一个单位。因为Linux是多用户多任务系统，所以一个文件可能同时被许多人使用，所以我们一定要设好每个文件的权限，其文件的权限位置排列顺序是（以-rwxr-xr-x为例）：
>　　rwx(Owner)r-x(Group)r-x(Other)
>　　这个例子表示的权限是：使用者自己可读，可写，可执行；同一组的用户可读，不可写，可执行；其它用户可读，不可写，可执行。另外，有一些程序属性的执行部分不是X,而是S,这表示执行这个程序的使用者，临时可以有和拥有者一样权力的身份来执行该程序。一般出现在系统管理之类的指令或程序，让使用者执行时，拥有root身份。
>　　第二个栏位，表示文件个数。如果是文件的话，那这个数目自然是1了，如果是目录的话，那它的数目就是该目录中的文件个数了。
>　　第三个栏位，表示该文件或目录的拥有者。若使用者目前处于自己的Home,那这一栏大概都是它的账号名称。
>　　第四个栏位，表示所属的组（group）。每一个使用者都可以拥有一个以上的组，不过大部分的使用者应该都只属于一个组，只有当系统管理员希望给予某使用者特殊权限时，才可能会给他另一个组。
>　　第五栏位，表示文件大小。文件大小用byte来表示，而空目录一般都是1024byte，你当然可以用其它参数使文件显示的单位不同，如使用ls –k就是用kb来显示一个文件的大小单位，不过一般我们还是以byte为主。
>　　第六个栏位，表示创建日期。以“月，日，时间”的格式表示，如Aug 15 5:46表示8月15日早上5:46分。
>　　第七个栏位，表示文件名。我们可以用ls –a显示隐藏的文件名。



| 命令（注意空格） | 解释                                                         |
| ---------------- | ------------------------------------------------------------ |
| cd /             | 切换当前目录至dirName                                        |
| cd ..            | 退到上层目录                                                 |
| cd ../.. //      | 进入当前目录的父目录的父目录。                               |
| mkdir            | 建立新的目录（如果目录存在会报错）                           |
| rmdir            | 删除已建立的目录， rmdir 命令只能删除空的文件夹，如果文件夹非空，将不能删除，它也没有-f选项，所以你的命令都是错的。要删除非空的文件夹，可以使用rm命令，加rf两个选项，如：`rm -rf sandao` |
| cp               | cp命令用来将一个或多个源文件或者目录复制到指定的目的文件或目录。它可以将单个源文件复制成一个指定文件名的具体的文件或一个已经存在的目录下。cp命令还支持同时复制多个文件，当一次复制多个文件时，目标文件参数必须是一个已经存在的目录，否则将出现错误。 |
|                  |                                                              |
|                  |                                                              |



>1. -a：此参数的效果和同时指定"-dpR"参数相同；  
>2. -d：当复制符号连接时，把目标文件或目录也建立为符号连接，并指向与源文件或目录连接的原始文件或目录；  
>3. -f：强行复制文件或目录，不论目标文件或目录是否已存在；  
>4. -i：覆盖既有文件之前先询问用户；  
>5. -l：对源文件建立硬连接，而非复制文件；  
>6. -p：保留源文件或目录的属性；   
>7. -R/r：递归处理，将指定目录下的所有文件与子目录一并处理；  
>8. -s：对源文件建立符号连接，而非复制文件；  
>9. -u：使用这项参数后只会在源文件的更改时间较目标文件更新时或是名称相互对应的目标文件并不存在时，才复制文件；  
>10. -S：在备份文件时，用指定的后缀“SUFFIX”代替文件的默认后缀；  
>11. -b：覆盖已存在的文件目标前将目标文件备份； -v：详细显示命令执行的操作  





如下，将/home/linlin1下的linlin.c复制到/home/linlin2下

![img](http://img.blog.csdn.net/20151018150534338)

如下，将/home/linlin1下的linlin.c复制到/home/linlin2下并改名为hello.c

![img](http://img.blog.csdn.net/20151018150846933)

我们在Linux下使用cp命令复制文件时候，有时候会需要覆盖一些同名文件，覆盖文件的时候都会有提示：需要不停的按Y来确定执行覆盖。文件数量不多还好，但是要是几百个估计按Y都要吐血了，于是折腾来半天总结了一个方法：

**[html]** [view plain](http://csdnimg.cn/release/phoenix/#) [copy](http://csdnimg.cn/release/phoenix/#) 

1. cp aaa/* /bbb 复制目录aaa下所有到/bbb目录下，这时如果/bbb目录下有和aaa同名的文件，需要按Y来确认并且会略过aaa目录下的子目录。  
2. cp -r aaa/* /bbb 这次依然需要按Y来确认操作，但是没有忽略子目录。  
3. cp -r -a aaa/* /bbb 依然需要按Y来确认操作，并且把aaa目录以及子目录和文件属性也传递到了/bbb。   
4. cp -r -a aaa/* /bbb 成功，没有提示按Y、传递了目录属性、没有略过目录。  

![img](http://img.blog.csdn.net/20151018151356664)

**rm**
　　这个命令是用来删除文件的，和dos下面的rm（删除一个空目录）是有区别的，大家千万要注意。rm命令常用的参数有三个： -i,-r,-f。

**[html]** [view plain](http://csdnimg.cn/release/phoenix/#) [copy](http://csdnimg.cn/release/phoenix/#) 

>1. -f, --force    忽略不存在的文件，从不给出提示。  
>2. -i, --interactive 进行交互式删除  
>3. -r, -R, --recursive   指示rm将参数中列出的全部目录和子目录均递归地删除。  
>4. -v, --verbose    详细显示进行的步骤  
>5. ​    --help     显示此帮助信息并退出  
>6. ​    --version  输出版本信息并退出  

　　rm –r 目录名：这个操作可以连同这个目录下面的子目录都删除，功能上和rmdir相似。
　　rm –f 文件名（目录名）：这个操作可以进行强制删除。 

如下，交互删除

![img](http://img.blog.csdn.net/20151018151802070)

全部强制删除

![img](http://img.blog.csdn.net/20151018151851402)

**mv**
　　这个命令的功能是移动目录或文件，引申的功能是给目录或文件重命名。当使用该命令来移动目录时，他会连同该目录下面的子目录也一同移走。

**[html]** [view plain](http://csdnimg.cn/release/phoenix/#) [copy](http://csdnimg.cn/release/phoenix/#) 

>1. -b ：若需覆盖文件，则覆盖前先行备份。   
>2. -f ：force 强制的意思，如果目标文件已经存在，不会询问而直接覆盖；  
>3. -i ：若目标文件 (destination) 已经存在时，就会询问是否覆盖！  
>4. -u ：若目标文件已经存在，且 source 比较新，才会更新(update)  
>5. -t  ： --target-directory=DIRECTORY move all SOURCE arguments into DIRECTORY，即指定mv的目标目录，该选项适用于移动多个源文件到一个目录的情况，此时目标目录在前，源文件在后。  



mv 原文件（目录）名 新的文件（目录）名。 

![img](http://img.blog.csdn.net/20151018152021031)

文件 移动

![img](http://img.blog.csdn.net/20151018152400986)

**du，df**
　　du命令可以显示目前的目录所占的磁盘空间，df命令可以显示目前磁盘剩余的磁盘空间。如果du命令不加任何参数，那么返回的是整个磁盘的使用情况，如果后面加了目录的话，就是这个目录在磁盘上的使用情况

查看当前目录以及子目录的大小:

![img](http://img.blog.csdn.net/20151018152913810)

查看整个磁盘大小

![img](http://img.blog.csdn.net/20151018153029956)

cat

​     cat主要有三大功能：
1.一次显示整个文件。$ cat filename
2.从键盘创建一个文件。$ cat > filename  
   只能创建新文件,不能编辑已有文件.
3.将几个文件合并为一个文件： $cat file1 file2 > file
参数：

**[html]** [view plain](http://csdnimg.cn/release/phoenix/#) [copy](http://csdnimg.cn/release/phoenix/#) 

1. -n 或 --number 由 1 开始对所有输出的行数编号  
2. -b 或 --number-nonblank 和 -n 相似，只不过对于空白行不编号  
3. -s 或 --squeeze-blank 当遇到有连续两行以上的空白行，就代换为一行的空白行  
4. -v 或 --show-nonprinting  

它的用法如下：

　　cat text 显示text这个文件；

　　cat file1 file2 依顺序显示file1,file2的内容；

![img](http://img.blog.csdn.net/20151018153427009)

把 text.c和 text1.c的档案内容加上行号（空白行不加）之后将内容附加到 text2.c 里。

![img](http://img.blog.csdn.net/20151018153638401)

![img](http://img.blog.csdn.net/20151018153828307)

**more,less**
　　这是两个显示一般文本文件的指令。如果一个文本文件太长了超过一个屏幕的画面，用cat来看实在是不理想，就可以试试more和less两个指令。More指令可以使超过一页的文件临时停留在屏幕，等你按任何的一个键以后，才继续显示。而less除了有more的功能以外，还可以用方向键往上或往下的滚动文件，所以你随意浏览，阅读文章时，less是个非常好的选择。 

more:选项 文件名

+n      从笫n行开始显示
-n       定义屏幕大小为n行
+/pattern 在每个档案显示前搜寻该字串（pattern），然后从该字串前两行之后开始显示  
-c       从顶部清屏，然后显示
-d       提示“Press space to continue，’q’ to quit（按空格键继续，按q键退出）”，禁用响铃功能
-l        忽略Ctrl+l（换页）字符
-p       通过清除窗口而不是滚屏来对文件进行换页，与-c选项相似
-s       把连续的多个空行显示为一行
-u       把文件内容中的下画线去掉

在每个档案显示前搜寻该字串lin，然后从该字串前两行之后开始显示 

![img](http://img.blog.csdn.net/20151018154140835)

**less**

less
 工具也是对文件或其它输出进行分页显示的工具，应该说是linux正统查看文件内容的工具，功能极其强大。less 的用法比起 more 
更加的有弹性。在 more 的时候，我们并没有办法向前面翻， 只能往后面看，但若使用了 less 时，就可以使用 [pageup] 
[pagedown] 等按键的功能来往前往后翻看文件，更容易用来查看一个文件的内容！除此之外，在 less 
里头可以拥有更多的搜索功能，不止可以向下搜，也可以向上搜。

less 选项 文件名

选项

**[html]** [view plain](http://csdnimg.cn/release/phoenix/#) [copy](http://csdnimg.cn/release/phoenix/#) 

1. -b <缓冲区大小> 设置缓冲区的大小  
2. -e  当文件显示结束后，自动离开  
3. -f  强迫打开特殊文件，例如外围设备代号、目录和二进制文件  
4. -g  只标志最后搜索的关键词  
5. -i  忽略搜索时的大小写  
6. -m  显示类似more命令的百分比  
7. -N  显示每行的行号  
8. -o <文件名> 将less 输出的内容在指定文件中保存起来  
9. -Q  不使用警告音  
10. -s  显示连续空行为一行  
11. -S  行过长时间将超出部分舍弃  
12. -x <数字> 将“tab”键显示为规定的数字空格  
13. /字符串：向下搜索“字符串”的功能  
14. ?字符串：向上搜索“字符串”的功能  
15. n：重复前一个搜索（与 / 或 ? 有关）  
16. N：反向重复前一个搜索（与 / 或 ? 有关）  
17. b  向后翻一页  
18. d  向后翻半页  
19. h  显示帮助界面  
20. Q  退出less 命令  
21. u  向前滚动半页  
22. y  向前滚动一行  
23. 空格键 滚动一行  
24. 回车键 滚动一页  
25. [pagedown]： 向下翻动一页  
26. [pageup]：   向上翻动一页  

![img](http://img.blog.csdn.net/20151018154917495)

退出按Q

向下搜索含lin的字符串

![img](http://img.blog.csdn.net/20151018155003026)

****

**tail**

​    也是用于用于查看文件内容 

tail语法格式： 
​    tail [ -f ] [ -c Number | -n Number | -m Number | -b Number | -k Number ] [ File ] 
​    或者 
​    tail [ -r ] [ -n Number ] [ File ] 
使用说明： 
​    tail 命令从指定点开始将 File 参数指定的文件写到标准输出。如果没有指定文件，则会使用标准输入。 Number 
变量指定将多少单元写入标准输出。 Number 变量的值可以是正的或负的整数。如果值的前面有 
+（加号），从文件开头指定的单元数开始将文件写到标准输出。如果值的前面有 
-（减号），则从文件末尾指定的单元数开始将文件写到标准输出。如果值前面没有 +（加号）或 -（减号），那么从文件末尾指定的单元号开始读取文件。 
主要参数： 
-b Number 从 Number 变量表示的 512 字节块位置开始读取指定文件。 
-c Number 从 Number 变量表示的字节位置开始读取指定文件。 
-f
 如果输入文件是常规文件或如果 File 参数指定 FIFO（先进先出），那么 tail 
命令不会在复制了输入文件的最后的指定单元后终止，而是继续从输入文件读取和复制额外的单元（当这些单元可用时）。如果没有指定 File 
参数，并且标准输入是管道，则会忽略 -f 标志。tail -f 命令可用于监视另一个进程正在写入的文件的增长。 
-k Number 从 Number 变量表示的1KB 块位置开始读取指定文件。 
-m Number 从 Number 变量表示的多字节字符位置开始读取指定文件。使用该标志提供在单字节和双字节字符代码集环境中的一致结果。 
-n Number 从首行或末行位置来读取指定文件，位置由 Number 变量的符号（+ 或 - 或无）表示，并通过行号 Number 进行位移。 
-r 从文件末尾以逆序方式显示输出。-r 标志的缺省值是以逆序方式显示整个文件。   
如果文件大于 20,480 字节，那么-r标志只显示最后的 20,480 字节。 -r 标志只有   与 -n 标志一起时才有效。否则，就会将其忽略。 

**tail -f 命令可用于监视另一个进程正在写入的文件的增长。 特别是在看日志时非常有用，你实时更新了日志，它就实时显示出来**

![img](http://img.blog.csdn.net/20151018155529829)

**pwd**
　　pwd [--help][--version]
　　说明：执行pwd指令可立刻得知您目前所在的工作目录的绝对路径名称。 

![img](http://img.blog.csdn.net/20151018155146546)

clear

**grep**
　　用于查找文件中符合字符串的哪行。
　　参数说明：

**[html]** [view plain](http://csdnimg.cn/release/phoenix/#) [copy](http://csdnimg.cn/release/phoenix/#) 

1.   -a ：将 binary 文件以 text 文件的方式搜寻数据  
2. -c ：计算找到 '搜寻字符串' 的次数  
3. -i ：忽略大小写的不同，所以大小写视为相同  
4. -n ：顺便输出行号  
5. -v ：反向选择，亦即显示出没有 '搜寻字符串' 内容的那一行!   

![img](http://img.blog.csdn.net/20151018160624870)

![img](http://img.blog.csdn.net/20151018160733089)

根据文件内容递归查找目录

**[html]** [view plain](http://csdnimg.cn/release/phoenix/#) [copy](http://csdnimg.cn/release/phoenix/#) 

1. \# grep ‘energywise’ *           #在当前目录搜索带'energywise'行的文件  
2. \# grep -r ‘energywise’ *        #在当前目录及其子目录下搜索'energywise'行的文件  
3. \# grep -l -r ‘energywise’ *     #在当前目录及其子目录下搜索'energywise'行的文件，但是不显示匹配的行，只显示匹配的文件  

![img](http://img.blog.csdn.net/20151018161013742)

**grep功能是很强大的，这里只简单说明了一下，有兴趣的同学自己下来研究下吧！**

**find**

**[html]** [view plain](http://csdnimg.cn/release/phoenix/#) [copy](http://csdnimg.cn/release/phoenix/#) 

1. $ find  -name "*.txt" -print 用于查找所有的‘ *.txt’文件在当前目录及子目录中  
2. $ find  -name "[A-Z]*" -print 用于当前目录及子目录中查找文件名以一个大写字母开头的文件  
3. $ find /etc -name "host*" -print 在/etc目录中查找文件名以host开头的文件  
4. $find  -name "[a-z][a-z][0--9][0--9].txt" -print 在当前目录查找文件名以两个小写字母开头，跟着是两个数字，最后是.txt的文件  

![img](http://img.blog.csdn.net/20151018161511584)

**[html]** [view plain](http://csdnimg.cn/release/phoenix/#) [copy](http://csdnimg.cn/release/phoenix/#) 

1. 1.在某目录下查找名为“elm.cc”的文件  
2. find /home/lijiajia/ -name elm.cc  
3.    ​
4. 2.查找文件名中包含某字符（如"elm"）的文件  
5. find /home/lijiajia/ -name '*elm*'  
6. find /home/lijiajia/ -name 'elm*'  
7. find /home/lijiajia/ -name '*elm'  
8.    ​
9. 3.根据文件的特征进行查询  
10. find /home/lijiajia/ -amin -10        #查找在系统中最后10分钟访问的文件  
11. find /home/lijiajia/ -atime -2        #查找在系统中最后48小时访问的文件  
12. find /home/lijiajia/ -empty           #查找在系统中为空的文件或者文件夹  
13. find /home/lijiajia/ -group cat       # 查找在系统中属于groupcat 的文件（试了，命令不对。）  
14. find /home/lijiajia/ -mmin -5         # 查找在系统中最后5 分钟里修改过的文件  
15. find /home/lijiajia/ -mtime -1        #查找在系统中最后24 小时里修改过的文件  
16. find /home/lijiajia/ -nouser          #查找在系统中属于作废用户的文件（不明白是什么意思）  
17. find /home/lijiajia/ -amin 10         #查找在系统中最后10分钟访问的文件  
18. find /home/ftp/pub -user lijiajia     #查找在系统中属于lijiajia这个用户的文件  
19. (PS:以上都是在 /home/lijiajia/文件夹下进行的操作)  
20.    ​
21. 4.使用混合查找方式查找文件  
22. find /tmp -size +10000000c -and -mtime +2      #查找/tmp目录中大于10000000字节并且在48小时内修改的某个文件  
23. find /tmp -user tom -or -user george           #查找/tmp目录中属于tom或者george这两个用户的文件  
24. find /tmp ! -usr fred                          #查找/tmp目录中不属于fred的文件  
25.    ​
26. 5.查找并显示文件  
27. find /home/lijiajia/ -name 'elm.cc' -ls        #在目录下查找名为“elm.cc”的文件,并显示这些文件的信息  