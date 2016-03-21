#<center>**Lab1实验报告**
####14231027 赵岳
##Chapter 1概述
通过本章的学习和实验，我初步了解了本学期OS实验的基本框架和一些实验所必须的基本技能。自己安装了虚拟机，但由于提供了远程虚拟机，所以最终还是选择在远程虚拟机上完成操作系统实验。同时，熟悉并了解了linux的一些基本操作和指令，为以后的实验打下了基础。
##Exercise 1.1
成功安装了Linux虚拟机并且设置账号密码进行登陆。与ssh远程虚拟机不同的是，安装的虚拟机具有图形化界面。  

<center>![本地虚拟机登陆界面](https://raw.githubusercontent.com/tigerzhaoyue/BUAA_MIPS_OS_DOC/master/Picture/Lab1/ubuntu%20denglu.JPG)  </center>
###ls 指令
ls 的意思是list，顾名思义，是列出当前目录下所有文件的意思。这条指令还含有多个参数，例如，ls -a即为列出目录下所有文件指令（包括以“.”为前缀的文件），这在实验初期也得到了验证。  
使用实例：  
`ls -a`
###cat 指令
cat 指令允许我们可以将一个文件内容显示到Terminal上，而不用使用文本编辑器打开之。这一指令使我们对一些文件进行修改后可以方便地进行查看是否保存成功，也可以让我们以只读的方式查看一些文件而避免误操作对文件的更改。  
使用实例：
`cat test.c`  
<center>![使用cat指令](https://raw.githubusercontent.com/tigerzhaoyue/BUAA_MIPS_OS_DOC/master/Picture/Lab1/cat.JPG)  </center>
###cd 指令
这条指令几乎是最常用的指令。由于没有图形化的操作界面，这就意味着我们在目录间进行切换时只能使用linux指令，即cd指令。根据自己的实验结果和查阅linux指令速查手册，得出了以下结论：  
 - `cd ..` 是回到上一级目录  
 - `cd /` 是回到根目录  
 - `cd .` 是当前目录  
 - `cd < foldername >`  是进入到该文件夹的目录下
###cp 指令
cp指令用于复制文件或目录，如同时指定两个以上的文件或目录，且最后的目的地是一个已经存在的目录，则它会把前面指定的所有文件或目录复制到此目录中.除此以外，该指令还提供了许多参数，用于进行各种有条件的复制。  
使用实例：    
`cp file1 file2`  
###mv 指令
mv 是move的缩写，是提供移动文件的功能的指令。经常用来备份文件和目录。同时，该指令还有一个作用就是文件重命名。  
使用实例：  
`mv a b` 可将文件名a改为b，文件的位置不发生变化。
`mv a b(a folder)` 可将文件a挪动到b文件夹去。    

<center>![使用mv指令](https://raw.githubusercontent.com/tigerzhaoyue/BUAA_MIPS_OS_DOC/master/Picture/Lab1/mv.JPG)  </center>    
##Exercise 1.2
由于本年度操作系统实验在远程虚拟机上完成，故未完成该部分实验。
##Thinking 1.1
###ls -l指令
ls指令加上-l参数后，显示的文件名以长格式形式显示，每个文件名占一行，包括读写权限，创建者，创建者所在组，和创建时间等详细信息。
###mv test1.c teset2.c
将test1.c 文件改名为test2.c。
###cp test1.c test2.c
创建一个test1.c的副本，并命名为test2.c。
###cd ..
回到上层目录。
##Thinking 1.2 
###grep简介
grep指令是"Globally search a Regular Expression and Print"的缩写。顾名思义，是指全局地使用正则匹配的方式搜索文本，并把匹配的行打印出来。  
其完整格式是`grep [-acinv] [--color=auto] '搜寻字符串' filename`其中"acinv"表示5个常用的参数，还可以给找到的关键词部分加上颜色，例如：  
<center>![使用grep](https://raw.githubusercontent.com/tigerzhaoyue/BUAA_MIPS_OS_DOC/master/Picture/Lab1/grep_color.JPG)  </center>  
###grep的5个常用参数 
1. -a 可以将二进制文件以text文件的方式搜索数据。本次实验里未用到，但以后会用。  
2. -c 计算找到搜索字符串的次数，例如：  
<center>![使用grep-c](https://raw.githubusercontent.com/tigerzhaoyue/BUAA_MIPS_OS_DOC/master/Picture/Lab1/grep_c.JPG)  </center>  
3. -i 查找时忽略大小写的不同。
4. -n 同时输出行号，而不仅是匹配的行的内容。例如：  
<center>![使用grep-c](https://raw.githubusercontent.com/tigerzhaoyue/BUAA_MIPS_OS_DOC/master/Picture/Lab1/grep_n.JPG)  </center>  
5. -v 反向选择，输出没有搜索字符串的行的内容。例如：  
<center>![使用grep-c](https://raw.githubusercontent.com/tigerzhaoyue/BUAA_MIPS_OS_DOC/master/Picture/Lab1/grep_v.JPG)  </center>   
###grep与正则表达式的配合使用
由于还涉及到正则表达式的用法，这里不再赘述，只举一个实验的例子。我们先新建一个含有两行英文的txt文件名为testgrep。内容为：`I love OS.I live in N625. `执行grep命令运行结果如下：  
<center>![使用grep-c](https://raw.githubusercontent.com/tigerzhaoyue/BUAA_MIPS_OS_DOC/master/Picture/Lab1/grep_regular.JPG)  </center>    
参考博客 ： [ggjucheng - 博客园](http://www.cnblogs.com/ggjucheng/archive/2013/01/13/2856896.html)  
  
  
 
***
  
##Chapter 2概述
本章，按照实验指导书的要求和顺序，了解了第一次实验的目的（成功加载linux内核到gxemul仿真器），结合理论课上王雷老师所讲的关于bootloader的知识，初步了解了机器开机加电后操作系统是如何被加载运行的。同时，在了解了一些相关文件的作用和原理，以及掌握如何使用文本编辑器后，对一些关键文件进行了改动，成功使内核正确运行。最后，还练习了强大的git工具，以及git的相关指令（以前只使用过图形化界面的git工具，虽然对git的status较为熟悉，但是对指令并不熟悉）。
##Exercise 2.1
关于修改include.mk的文件，实质上是修改makefile文件里的内容，根据指导书上的内容，makefile是“内核代码的地图”，make工具会读取makefile文件，并以此为依据编译和维护工程。在我们的实验里，为了编译出vmlinux可执行文件，就必须使用make指令，而makefile里的交叉编译器位置有误，故我们需要修改交叉编译器的路径，使之成为正确的路径。
最终，修改路径为`/OSLAB/compiler/usr/bin/mips_4KC_gcc`即可。  
至此，我们的交叉编译器路径已经正确，可以成功执行make指令，生成了vmlinux文件，完成。
##Exercise 2.2
进行该部分实验前，根据指导书的介绍，又参考了网上[GUN LD Simple Linker Script Example](https://www.sourceware.org/binutils/docs/ld/Simple-Example.html#Simple-Example)的内容，知道了可以用Linker Script增强连接器的通用性，使得它可以为不同的平台生成可执行文件。Linker Script中记录了各个section该如何被映射到segment上，又该加载到何处。  
在完成2.2之前，还完成了一个小实验：用Linker Script修改text、data和bss的位置.如图：  
<center>![使用readelf](https://raw.githubusercontent.com/tigerzhaoyue/BUAA_MIPS_OS_DOC/master/Picture/Lab1/readelf.JPG)  </center>  
有了这些基础后，我们只需知道内核的代码段需要被加载的地址，便可以利用Linker Script来修改内核加载的位置了。根据32 bit Mips CPU的特性，我们需要一个地址可直接转化为物理地址的区段，并且通过cache进行存取的区段。经过选择，只能选择seg0段。读取mmu.h的相关内容后，最终确定位置为Kernel Text段，即0x80010000。有了这些知识，我们便可以进行修改了。最终添加SECTION的相关内容到scse0_3.lds即可。
##Exercise 2.3
结合上学期计算机组成原理的相关知识（或指导书中关于MIPS汇编的相关部分），我们知道最终汇编语言调用函数实际上分为减小sp指针，为栈分配空间，增加sp指针，释放栈空间这几步。可见sp指针有着非常强大的控制作用。我们为了正确地运行内核，就需要设置好栈指针。结合mmu.h内的图片，我们知道栈的指针应设在0x80400000位置。完成start.S后，运行  
`/OSLAB/gxemul -E testmips -C R3000 -M 64 /home/14231027/14231027-lab/gxemul/vmlinux` 指令    
可以成功跳转到main函数了。实验现象如下：  
<center>![使用readelf](https://raw.githubusercontent.com/tigerzhaoyue/BUAA_MIPS_OS_DOC/master/Picture/Lab1/mainstart.JPG)  </center>    
这里有两点需要注意：由于我们的交叉编译器在OSLAB目录下，所以要写`/OSLAB/gxemul`（不同于指导书）。在修改start.S时，需要把加入的一行代码放在死循环前面（或把死循环删去，目前看来这种做法暂时没有什么副作用），否则无法正确跳转到main函数。
##Exercise 2.4
##Exercise 2.5
##Exercise 2.6
##Exercise 2.7
##Thinking 2.2
##Exercise 2.8
***
#总结
###小建议
在做到实验2.2时感觉难度突然增高，许多地方读懂了实验手册，却由于不熟悉指令无从下手，许多同学（包括我）还利用了搜索引擎去了解操作的步骤。我个人认为该Lab1重点在理解我们是在干什么，而不是如何快速地操作linux，所以希望在实验手册中（至少Lab1中）加入每一步操作的指令（或者给一个详细操作的例子），这样不仅能使实验手册更具亲和力，也能让同学们更快地完成实验目的，获得满足感，对以后的实验充满信心。