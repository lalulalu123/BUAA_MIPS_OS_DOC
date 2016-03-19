#<center>Lab1实验报告
####14231027 赵岳
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
  
  
 
  
  
  
---  


