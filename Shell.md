[toc]



# Shell

## 定义

Shell是在Linux内核与用户之间的解释器程序，通常指的是bash，负责向内核翻译及传达用户/程序指令。

```shell
[root@sv7 ~] cat /etc/shells       #查看所有解释器
/bin/sh
/bin/bash
/usr/bin/sh
/usr/bin/bash

[root@svr7 ~] sh         #切换成sh解释器
sh-4.2# ls            #利用sh解释器输入命令
sh-4.2#exit           #退出sh解释器

[root@sv7 ~] useradd -s /bin/ksh tom       #指定用户登录的解释器
[root@sv7 ~] grep tom /etc/passwd
tom:x:1000:1000::/home/tom:/bin/ksh
```

bash可实现：Tab键、快捷键、历史命令、支持别名、管道、重定向。

验证重定向

\> 重定向标准输出

2\>   重定向错误输出

&\>  重定向所有输出

```shell
#错误输出到文件
[root@sv7 ~] ls xx
ls: cannot access 'xx': No such file or directory
[root@sv7 ~] ls xx 2> log1     
[root@sv7 ~] cat log1 
ls: cannot access 'xx': No such file or directory

#所有输出（标准输出和错误输出）到文件
[root@sv7 ~] ls a.txt xx
ls: cannot access 'xx': No such file or directory
a.txt
[root@sv7 ~] ls a.txt xx &> log2
[root@sv7 ~] cat log2 
ls: cannot access 'xx': No such file or directory
a.txt
```

## Shell变量

### 自定义变量

等号两边不能有空格，变量名称可以用数字、字母、下划线，不能以数字开头，不能用特殊符号

```shell
1）新建/赋值变量
[root@sv7 ~] x=10    #等号两边不能有空格，变量名称可以用数字、字母、下划线，不能以数字开头，不能用特殊符号

2）查看变量（调用变量），通过echo $变量名 可输出变量值：
[root@sv7 ~] echo $x
10

变量名称可以用数字、字母、下划线，不能以数字开头，不能用特殊符号
[root@sv7 ~] a=1
[root@sv7 ~] echo $a
1
[root@sv7 ~] a1=2
[root@sv7 ~] echo $a1
2
[root@sv7 ~] a?1=2
-bash: a?1=2: command not found
[root@sv7 ~] 1a=2
-bash: 1a=2: command not found
查看变量时，若变量名称与后面要输出的字符串连在一起，则应该以{}将变量名括起来以便区分：
[root@sv7 ~] echo $xRMB       #无法识别变量名x
[root@sv7 ~] echo ${x}RMB      #使用{}可以防止与后续字符混淆
10RMB

3）撤销自定义变量，若要撤销已有的变量，可将变量赋值为空或使用unset命令：
[root@sv7 ~] unset x        #撤销变量x
[root@sv7 ~] echo $x        #查看时已无结果
```

### 环境变量

使用环境变量，当前用户的环境变量USER记录了用户名、HOME记录了家目录、SHELL记录了登录解释器、HOSTNAME记录主机名、UID是用户的id号：

```shell
[root@sv7 ~] echo $USER $HOME $SHELL $UID
root /root /bin/bash 0
[root@sv7 ~] echo $HOSTNAME
sv7
```

可以把变量放入/etc/profile，对所有用户有效；放入~/.bash_profile，仅对指定的用户有效，使用env可查看所有环境变量，使用set可查看所有变量

### 位置变量

``` shell
#!/bin/bash
echo $0                   #脚本的名称
echo $1                   #第一个参数
echo $2                   #第二个参数
echo $*                   #所有参数
echo $#                   #所有参数的个数(参数有几个)
echo $$                   #执行脚本的进程号（或者说当前进程的进程号）
echo $?            		  #上一命令的返回状态码，0为成功，非0为异常
```

### 拓展应用

双引号的应用，使用双引号可以界定一个完整字符串，在命令中是使用时可以使命令读取到特殊变量和字符

```shell
[root@sv7 ~] x=a b c
-bash: b: command not found          #未界定时赋值失败
[root@sv7 ~] x="a b c"           #界定后成功
[root@sv7 ~] echo $x
a b c
```

单引号的应用，界定一个完整的字符串，并且可以实现屏蔽特殊符号的功能

```shell
[root@sv7 ~] test=11
[root@sv7 ~] echo '$test'
$test
```

反撇号或`$()`的应用，使用反撇号或`$()`时，可以将命令执行的结果作为字符串存储，因此称为命令替换

```shell 
[root@sv7 ~] a=`date`  #将date执行结果赋值给a
[root@sv7 ~] a=$(date)  #效果同上
[root@sv7 ~] echo $a
Sun Nov 3 17:30:53 CST 2024
[root@sv7 ~] touch `date +%F`.txt
[root@sv7 ~] ls 
 2024-11-03.txt 
案例：使用tar命令把etc目录备份，备份时加上日期
[root@sv7 ~] tar -czf etc-`date +%F`.tar.gz /etc
[root@sv7 ~] ls 
etc-2024-11-03.tar.gz
```

使用read命令从键盘读取变量值（只定义变量，不赋值，需要自行输入具体的值）
1）read基本用法，执行后从会等待并接受用户输入（无任何提示的情况），并赋值给变量str：

```shell
[root@sv7 ~] read str
123            #随便输入一些文字，按回车键提交
[root@sv7 ~] echo $str       #查看赋值结果
123
```

为了不至于使用户不知所措、莫名其妙，推荐的做法是结合-p选项给出友好提示：

```shell
[root@sv7 ~] read -p "请输入一个整数：" i
请输入一个整数：240
[root@sv7 ~] echo $i
240
```

2）此时的脚本在设置密码时，密码能看到，不安全；可以使用stty终端显示控制，将回显功能关闭（`stty -echo`），将回显功能恢复（`stty echo`）

```shell
stty -echo
read -p "请输入密码: " pass
stty echo
```

### 全局变量

使用export发布全局变量，默认情况下，自定义的变量为局部变量，只在当前Shell环境中有效，而在子Shell环境中无法直接使用。

```shell
若希望定义的变量能被子进程使用：
[root@sv7 ~] export a=11       #使用export定义全局变量，如果是已经存在的局部变量要定义为全局变量执行 export 变量名  即可
[root@sv7 ~] echo $a           #查看a变量的值，有值
验证刚刚发布的全局变量：
[root@sv7 ~] bash                #进入bash子Shell环境
[root@sv7 ~] echo $a             #查看全局变量的值
[root@sv7 ~] exit              #返回原有Shell环境
[root@sv7 ~] unset a 			#删除变量
```

### 数值运算

1、使用$(( ))、$[ ]、let等整数运算工具：进行四则运算及求模结果
2、使用bc实现小数运算操作
使用`$(())`或`$[]`表达式，引用变量可省略 $ 符号；计算结果替换表达式本身，可结合`echo`命令输出

```shell
[root@sv7 ~] echo $[10%3]
1
[root@sv7 ~] echo $((3+2))
5
```

变量自增/自减操作，使用let命令，let命令可以直接对变量值做运算再保存新的值。

```shell
[root@sv7 ~] i=10
[root@sv7 ~] let i++
[root@sv7 ~] echo $i
11
```

### 字符串处理

```shell
字符串截取的用法： ${变量名:起始位置:长度}
起始位置从0开始计数

字符串替换的两种用法：
1、只替换第一个匹配结果：${变量名/old/new}
2、替换全部匹配结果：${变量名//old/new}

字符串掐头去尾：
1、从左向右，最短匹配删除：${变量名#*关键词}
2、从左向右，最长匹配删除：${变量名##*关键词}
3、从右向左，最短匹配删除：${变量名%关键词*}
3、从右向左，最长匹配删除：${变量名%%关键词*}
注：以上操作都不会修改源数据

字符串初值
只取值，${var:-word}，若变量var已存在且非空，则返回 $var 的值；否则返回字串“word”，原变量var的值不受影响。

[root@sv7 ~] vim useradd.sh
#!/bin/bash
read -p  "请输入用户名:"  user
[ -z $user ] && exit         #如果无用户名，则脚本退出
read -p  "请输入密码:"  pass
useradd $user
echo ${pass:-123456} | passwd  --stdin  $user   #如果用户没有输入密码，则默认密码为123456
[root@sv7 ~] chmod +x useradd.sh 
[root@sv7 ~] ./useradd.sh 
请输入用户名:nacy         #输入用户名
请输入密码:               #直接回车
Changing password for user nacy.
passwd: all authentication tokens updated successfully.
```

## 条件测试操作

语法格式：使用 “test 表达式” 或者 [ 表达式 ] 都可以，注意空格不要缺少。
条件测试操作本身不显示出任何信息。所以可以在测试后查看变量`$?`的值来做出判断。

### 字符串比较

判断变量是否为空，格式 \[ 表达式 字符串 ] 
\-z  字符串为空
\-n  字符串不为空

```shell
[root@sv7 ~] a=22
[root@sv7 ~] [ -z "$a" ]   #判断$a为空
[root@sv7 ~] echo $?       #结果错误，$a不为空，变量中有值
1
```

一行执行多条命令

```shell
# A ; B            #执行A命令后执行B命令，两者没有逻辑关系
# A && B           #仅当A命令执行成功，才执行B命令，&&并且的意思，执行失败则不往下执行
# A || B           #仅当A命令执行失败，才执行B命令，||或者的意思，A执行成功则不执行B，A执行失败，则执行B
```

字符串是否相等

\[  字符串1  操作符   字符串2  ]

== 两个字符串相等

!= 两个字符串不相同

```shell
[root@sv7 ~] a=hello
[root@sv7 ~] b=hello
[root@sv7 ~] [ "$a" == "$b" ] && echo Y || echo N
Y
!= 两个字符串不相同
[root@sv7 ~] [ "$a" != "$b" ] && echo Y || echo N
N
```

整数比较（参与比较的必须是整数）
\-gt 大于  
\-ge 大于或等于
\-lt  小于
\-le 小于或等于
\-eq 等于
\-ne 不等于

### 文件状态测试

| 操作符 | 含义                                         |
| :----: | -------------------------------------------- |
|   -e   | 判断对象是否存在（Exit），存在结果为真       |
|   -d   | 判断对象是否为目录（Directory），是则为真    |
|   -f   | 判断对象是否为一般文件（File），是则为真     |
|   -r   | 判断对象是否有可读（Read），是则为真         |
|   -w   | 判断对象是否有可写（Write）权限，是则为真    |
|   -x   | 判断对象是否有可执行（Excute）权限，是则为真 |

## 结构化命令

### if-then语句

语法结构

```shell
#if单分支的语法组成：
if 条件测试;then 
    命令序列
fi

#if双分支的语法组成：
if 条件测试;then
    命令序列1
else 
    命令序列2
fi

#if多分支的语法组成：
if  条件测试1 ;then 
    命令序列1
elif 条件测试2 ;then 
    命令序列2
else
    命令序列n
fi
```

### for循环结构

语法结构

```shell
for 变量名 in 值1 值2  #值的数量决定循环任务的次数
do
	命令序列
done
```

具体案例

```shell
[root@sv7 ~] for i in 1 2 3 4 5
do
   echo hello
done

[root@sv7 ~] for i in {1..10} #花括号内不能使用变量
do
   echo "abc"
done

[root@sv7 ~] for i in $(seq 10)   #seq 可以接受参数变量
do
   echo "abc"
done
```

### while循环结构

while循环属于条件式的执行流程，会反复判断指定的测试条件，只要条件成立即执行固定的一组操作，直到条件变化为不成立为止。所以while循环的条件一般通过变量来进行控制，在循环体内对变量值做相应改变，以便在适当的时候退出，避免陷入死循环。

语法结构

```shell
使用while循环，语法结构如下所示：
while 条件测试    #根据条件的结果决定是否要执行任务，条件测试成功的话就执行，如果失败立刻结束循环
do
  命令序列
done
```

具体案例

```shell
练习while循环基本用法
-le小于等于
有限的循环，每次循环+1
[root@sv7 ~] i=1
[root@sv7 ~] while [ $i -le 5 ]
 do
    echo hello
    let i++
 done

[root@sv7 ~] vim run.sh 
#!/bin/bash
num=$[RANDOM%10+1]
while :
do
        read -p "我有一个1-10的数字，你猜是啥：" cai
        if [ $cai -eq $num ];then
          echo "猜对了"
          exit                  #添加exit，猜对了之后退出脚本
        elif [ $cai -gt $num ];then
          echo "猜大了"
        else
          echo "猜小了"
        fi
done
```

### case分支

case分支属于匹配执行的方式，它针对指定的变量预先设置一个可能的取值，判断该变量的实际取值是否与预设的某一个值相匹配，如果匹配上了，就执行相应的一组操作，如果没有任何值能够匹配，就执行预先设置的默认操作。
语法结构

```shell
case 变量 in
模式1)
  命令序列1 ;;
模式2)
  命令序列2 ;;
  .. ..
*)
  默认命令序列
esac	
```

## Shell函数

在Shell脚本中，将一些需重复使用的操作，定义为公共的语句块，即可称为函数。通过使用函数，可以使脚本代码更加简洁，增强易读性，提高Shell脚本的执行效率

语法结构

```shell
格式1：
function 函数名 {
  命令序列
  .. ..
}

格式2：
函数名() {
  命令序列
  .. ..
}
```

函数调用
直接使用“函数名”的形式调用，如果该函数能够处理位置参数，则可以使用“函数名 参数1 参数2 .. ..”的形式调用。
注意：函数的定义语句必须出现在调用之前，否则无法执行。
案例

```shell
步骤一：编写mycolor.sh脚本
1）任务需求及思路分析
用户在执行时提供2个整数参数，这个可以通过位置变量$1、$2读入。
调用函数时，将用户提供的两个参数传递给函数处理。
颜色输出的命令:echo -e "\033[32mOK\033[0m"； 
3X为字体颜色，4X为背景颜色，9x为字体高亮色，"\033[43;31mOK\033[0m"可以同时修改背景与字体颜色

2）编写shell脚本，利用函数与echo指令，输出彩色字体
[root@sv7 ~] vim mycolor.sh
#!/bin/bash
cecho() {
  echo -e "\033[$1m$2\033[0m"
}
cecho 32 OK
cecho 33 OK
cecho 34 OK
cecho 35 OK
```



## 正则表达式

### 基本正则列表

| 正则符号  | 描述                                     |
| :-------: | ---------------------------------------- |
|     ^     | 匹配行首                                 |
|    \$     | 匹配行尾                                 |
|    \[]    | 集合，匹配集合中的任意单个字符           |
|   \[^]    | 对集合取反                               |
|     .     | 匹配任意单个字符                         |
|    \*     | 匹配前一个字符任意次数 [*不允许单独使用] |
| \\{n,m\\} | 匹配前一个字符n到m次                     |
|  \\{n\\}  | 匹配前一个字符n次                        |
| \\{n,\\}  | 匹配前一个字符n次及以上                  |
|  \\(\\)   | 组合为整体，保留                         |

### 扩展正则列表

| 正则符号 | 描述                 |
| :------: | -------------------- |
|    +     | 最少匹配一次         |
|    ？    | 最多匹配一次         |
|  {n,m}   | 匹配前一个字符n到m次 |
|    ()    | 组合为整体，保留     |
|    \|    | 或者                 |
|    \b    | 单词边界             |

### 具体案例

```shell
[root@sv7 ~] grep "[a-z]" user       #找所有小写字母
[root@sv7 ~] grep "[A-Z]" user       #找所有大写字母
[root@sv7 ~] grep "[a-Z]" user       #找所有字母
[root@sv7 ~] grep "[^0-9a-Z]" user     #找所有符号
[root@sv7 ~] grep "r..t" user      #找rt之间有2个任意字符的行
[root@sv7 ~] grep "r.t" user       #找rt之间有1个任意字符的行，有匹配就输出；没有匹配就无输出
[root@sv7 ~] grep "o\{1,2\}" user              #找o，可以匹配1~2次
[root@sv7 ~] echo rooooot | grep "o\{3,\}"     #找o，可以匹配3个以及以上，都会匹配
[root@sv7 ~] echo rooooot | grep "o\{3\}"      #找o，只匹配3个

[root@sv7 ~] echo ababab | grep "\(ab\)\{3,5\}"    #匹配ab 3~5次，小括号的作用是将字符组合为一个整体
ababab 
```

扩展正则表达式，以上命令均可以加-E选项并且去掉所有\，-E使用扩展正则的用法

```shell
[root@sv7 ~] echo ababab | grep -E "(ab){3,5}"
ababab

测试\b 单词边界
[root@sv7 ~] echo iwanttheapple | grep "the"   #可以过滤出the
iwangttheapple
[root@sv7 ~] echo iwanttheapple | grep -E "\bthe"  #结果the不出来，the前面不能有内容，需要是一个独立的the才能出来

[root@sv7 ~] echo iwant theapple | grep -E "\bthe" #字符串the前面加空格
iwant theapple
[root@sv7 ~] echo iwant theapple | grep -E "\bthe\b"   #后面加\b结果也是出不来，the后面也能不能有内容
[root@sv7 ~] echo iwant the apple | grep -E "\bthe\b"  #the后面加空格，才能出来
iwant the apple
```

## 中断及退出

1、exit结束循环以及整个脚本
2、break可以结束整个循环
3、continue结束本次循环，进入下一次循环


## sed基本用法

sed文本处理工具的用法：
用法1：前置命令 | sed [选项] '条件指令'
用法2：sed [选项] '条件指令' 文件.. ..

sed命令的常用选项如下：

- -n（屏蔽默认输出，默认sed会输出读取文档的全部内容）
- -r（支持扩展正则）
- -i（修改源文件）

条件可以是行号或者/正则/，没有条件时默认为所有行都执行指令，指令可以是p输出、d删除、s替换
案例

```shell
[root@sv7 ~] sed -n 'p' user               #输出所有行
[root@sv7 ~] sed -n '1p' user              #输出第1行
[root@sv7 ~] sed -n '2p' user              #输出第2行

[root@sv7 ~] sed -n '2,4p' user            #输出2~4行
[root@sv7 ~] sed -n '2p;4p' user           #输出第2行与第4行
[root@sv7 ~] sed -n '3,+1p' user           #输出第3行以及后面1行
[root@sv7 ~] sed -n '1~2p' user            #输出奇数行

特殊用法
[root@sv7 ~] sed -n '1!p' user             #输出除了第1行的内容，!是取反
[root@sv7 ~] sed -n '$p' user              #输出最后一行
[root@sv7 ~] sed -n '=' user               #输出行号，如果是$=就是最后一行的行号
以上操作，如果去掉-n,再将p指令改成d指令就是删除

[root@sv7 ~] sed 's/2017/6666/' shu.txt     #把所有行的第1个2017替换成6666
[root@sv7 ~] sed 's/2017/6666/g' shu.txt    #所有行的所有2017都替换成6666
[root@sv7 ~] sed '2s/2017/6666/g' shu.txt   #第2行的所有2017都替换成6666

[root@sv7 ~] sed 's/2017/6666/2' shu.txt    #把所有行的第2个2017替换成6666
[root@sv7 ~] sed '1s/2017/6666/' shu.txt    #把第1行的第1个2017替换成6666
[root@sv7 ~] sed '3s/2017/6666/3' shu.txt   #把第3行的第3个2017替换成6666
[root@sv7 ~] sed '/2024/s/2017/6666/g' shu.txt #找含有2024的行，将里面的所有2017替换成6666

[root@sv7 ~] sed 's,/bin/bash,/sbin/sh,' user      #最佳方案，更改s的替换符；s后面跟/，/就是替换符；s后面跟！，！就是替换符；s后面跟，那么，就是替换符
```

**更该s的替换符**

```shell
[root@sv7 ~] sed 's,/bin/bash,/sbin/sh,' user      #最佳方案，更改s的替换符；s后面跟/，/就是替换符；s后面跟！，！就是替换符；s后面跟，那么，就是替换符
```

### 扩展

sed中的a、i、c

```shell
a：append 行后追加
i：insert 行前插入
c：替换，替换一行内容
[root@sv7 ~] cat user
root:x:0:0:root:/root:/bin/bash
[root@sv7 ~] sed '1a xx' user
root:x:0:0:root:/root:/bin/bash
xx
[root@sv7 ~] sed '1i xx' user
xx
root:x:0:0:root:/root:/bin/bash
[root@sv7 ~] sed '1c xx' user
xx
```

## awk使用

### awk基本用法

格式1：awk  [选项]  '[条件\]  {指令}'   文件
格式2：前置指令  |  awk \[选项\]   \' \[条件]  \{指令} \'
其中，print是最常用的指令；若有多条编辑指令，可用分号分隔。

Awk过滤数据时支持仅打印某一列，如第2列、第5列等。
处理文本时，默认将空格、制表符作为分隔符。
条件可以用/ /的方式，与sed类似

```shell
awk常用内置变量：
$0   文本当前行的全部内容
$1   文本的第1列
$2   文件的第2列
$3   文件的第3列，依此类推
NR   文件当前行的行号
NF   文件当前行的列数（有几列）

[root@sv7 ~] awk '/^root/{print NR}' user      #找以root开头的行，显示该行的行号
[root@sv7 ~] awk'/^root/{print NR,$0}' user   #找以root开头的行，显示该行的行号，输出该行的所有内容

选项 -F 可指定分隔符
[root@sv7 ~] awk -F: '{print $1}' user         #文档中如果没有空格，可以用F修改分隔符
[root@sv7 ~] awk -F: '{print $1,$6}' user      #使用冒号作为列的分隔符，显示第1、6列
```

awk处理的时机
awk会逐行处理文本，支持在处理第一行之前做一些准备工作，以及在处理完最后一行之后做一些总结性质的工作。在命令格式上分别体现如下：
awk [选项] '[条件]{指令}' 文件
awk [选项] 'BEGIN{指令} {指令} END{指令}' 文件
  BEGIN{ }    行前处理，读取文件内容前执行，指令执行1次
  { }         逐行处理，读取文件过程中执行，指令执行n次
  END{ }     行后处理，读取文件结束后执行，指令执行1次

```shell
[root@sv7 ~] awk -F: 'BEGIN{print "用户名 UID 家目录"}  {print $1,$3,$6}  END{print "系统中 有:",NR,"个用户"}' /etc/passwd | column -t
用户名            UID    家目录
root              0      /root
bin               1      /bin
daemon            2      /sbin
```

### awk处理条件

1. 使用正则设置条件
   /正则/   ~包含  !~不包含

   ```shell
   [root@sv7 ~] awk '/root/{print}' user  #找含有root的行打印
   [root@sv7 ~] awk -F: '$1~/root/{print}' user    #输出第1列包含root的行
   ```

2. 使用数值/字符串比较设置条件
   ==（等于）  !=  >  >=\(大于等于\)  <  <=

3. 逻辑测试条件; &&并且 ； || 或者

   ```shell
   [root@sv7 ~] awk -F: '$3>10&&$3<15{print}' user        #找第3列uid大于10 小于15的行
   [root@sv7 ~] awk -F: 'NR==2||NR==4{print}' user        #找行号是2或者4的行
   ```

4. 数学运算，BEGIN可以单独使用，不依赖文件，后面可以不跟文件名

   ```shell
   [root@sv7 ~] awk -F: '$3>10&&$3<15{print}' user        #找第3列uid大于10 小于15的行
   [root@sv7 ~] awk -F: 'NR==2||NR==4{print}' user        #找行号是2或者4的行
   ```

### awk数组

数组是一个可以存储多个值的变量，具体使用的格式如下：
定义数组的格式：数组名[下标]=元素值
调用数组的格式：数组名[下标]
注意：awk数组的下标除了可以使用数字，也可以使用字符串，字符串需要使用双引号：

```shell
for(变量名 in 数组名){print 变量名}  #这个格式可以查看数组的所有下标
[root@sv7 ~] awk 'BEGIN{a["abc"]="tom";a["xyz"]="jim";for(i in a){print i}}'       #此时取的是a的下标，如果想要取值可以使用下面命令
abc
xyz
```

案例

```shell
1）提取IP地址及访问量,统计每个人访问了多少次
[root@sv7 ~]@ vim log.txt
192.168.88.5 /a.html
192.168.88.5 /c.png
192.168.77.5 /x.php
192.168.88.9 /a.mp3
192.168.160.5 /u.doc
172.16.88.5 /e.html
[root@sv7 ~] awk '{ip[$1]++}END{for(i in ip){print ip[i],i }}' log.txt   #数组名称可以自定义其他的，通过awk数组+for循环查看日志中哪个ip来访过以及来访的次数

2）对第1）步的结果根据访问量排名
[root@sv7 ~] awk '{ip[$1]++}END{for(i in ip){print ip[i],i}}' log.txt | sort -nr  #使用sort命令增加排序功能，-n是以数字形式排序，-r是降序
```

## 案例

### 系统初始化

要求
某企业准备了一批Linux服务器(系统有7版本与8版本)来运行业务，现在需要将服务器做初始配置，编写一个脚本可以匹配不同系统的服务器实现以下需求：
1、所有服务器永久关闭防火墙服务和SELinux
2、关闭7版本系统的命令历史记录，修改8版本的命令历史记录最多保存2000条并加上时间戳  
3、关闭8版本系统的交换分区
4、定义root远程登录系统后的ssh保持时间为300秒
5、设置时间同步，ntp服务器地址是192.168.88.240

编写shell脚本，关闭防火墙，利用sed永久关闭selinux，定义root远程登录系统后的ssh保持时间为300秒，并设置时间同步，ntp服务器地址是192.168.88.240，然后判断linux系统版本，关闭7版本系统的命令历史记录，修改8版本的命令历史记录最多保存2000条并加上时间戳，关闭8版本系统的交换分区

```shell
vim init01.sh
#!/bin/bash
#脚本执行完后，用ssh远程登录测试
#可以先手工备份/etc/fstab和/etc/profile

#1)判断当前账户身份，并关闭防火墙与selinux
[ $UID -ne 0 ] && echo "请使用管理员操作" && exit
systemctl  stop firewalld
systemctl disable firewalld
setenforce 0
sed -i 's/SELINUX=enforcing/SELINUX=disabled/'  /etc/selinux/config
egrep -q "8\.[0-9]"  /etc/redhat-release      #判断系统版本
if [ $? -eq 0 ];then
	sed -i '/HISTSIZE=/c HISTSIZE=2000' /etc/profile  #历史命令2000条
	sed -i '/HISTSIZE=/a HISTTIMEFORMAT="%F %T "'  /etc/profile  #添加历史命令时间戳
	swapoff -a  			#关闭交换分区
	sed -i '/swap/s/^/#/' /etc/fstab			#关闭交换分区自动挂载
else
	sed -i 'HISTSIZE=/c HISTSIZE=0'   /etc/profile    #关闭历史命令
fi

echo "export TMOUT=300" >> ~/.bash_profile       #定义ssh超时时间
yum -y install chrony &> /dev/null
systemctl restart chronyd

```

### 文档处理

< 符号，输入重定向，可以在后面需要跟文件名，让程序不再从键盘读取数据，而是从文件中读取数据。

```shell
[root@sv7 ~] yum -y install postfix mailx
[root@sv7 ~] systemctl start postfix
[root@sv7 ~] mail -s test root     #ctrl +d 结束
123
456
[root@sv7 ~] mail  #查看邮件
写个脚本发邮件，可以用到< 输入重定向,往里面导入内容
[root@sv7 ~] vim mail.txt
hello linux
[root@sv7 ~] mail -s test root < mail.txt  #此时已经成功从文件读取数据，不用在自己输入
```

<< 符号也可以导入数据，代表你需要的内容在这里，某指令导入字符串时使用，而无需文件；EOF代表从这开始，最后的EOF代表结束；成对出现，也可以用其他的符号，EOF没有任何含义，可以随便改

```shell
[root@sv7 ~] mail -s test root << EOF
hello
test mail~
EOF

#使用read指令配合输入重定向可以同时定义多个变量
[root@sv7 ~] read a b < mail.txt   #利用mail.txt文档内容赋值，仅读取第一行
[root@sv7 ~] echo $a
hello
[root@sv7 ~] echo $b
linux
#结合while循环批量读取数据通过read命令给变量赋值
[root@sv7 ~] while read a b 
do
  echo $a $b
done < mail.txt      #利用mail.txt文档内容赋值，读取所有行
```

### 通过文档批量创建用户并配置密码

系统中的/dev/urandom可以获得取之不尽的随机字符，但内容太随意有些是不需要的，如果文档中没有密码，可以使用tr处理这些随机字符获取密码

```shell
[root@sv7 ~]~ strings /dev/urandom  #strings类似于cat，看这个urandom中的内容

[root@sv7 ~]~ tr -cd '_a-zA-Z0-9' < /dev/urandom | head -c 10   #-c是取反 -d是删除，删除的是从/dev/urandom中导入的数据；也就是删除的是_a-zA-Z0-9；但是-c是取反，对_a-zA-Z0-9取反删除，即就只是_a-zA-Z0-9这个范围内的字符串，head -c 10 可以得到10个字符
```



编写shell脚本，循环处理user.txt文档每行，跳过没有英文的行（如研发部），创建用户的同时使用tr与urandom文件生成10位随机密码利用sed写在user.txt文档的用户名后（第二列），并给用户配置此密码，之后行尾追加“已创建”三个字写入user.txt文档（此处将user.txt内容一起提交）

```shell
[root@sv7 ~] vim user.txt      
研发部
zhangsan   
人事部
lisi     
wangwu    
销售部
zhaoliu

#编写脚本
vim user02.sh
#!/bin/bash
x=`awk '/^[a-zA-Z0-9] && !/已创建/{print NR}' user.txt`  #找到用户是以数字或者字母开头并且不是已经创建的用户，打印行号
if [ -z $x ];then  	#判断x是否为空，是则无新用户需要创建
	echo "没有新用户需要创建"
	column -t user.txt
	exit
fi

for i in $x
do	
	p=$(tr -cd '_a-zA-Z0-9' < /dev/urandom | head -c 10)  #创建随机密码
	sed -i "${i}s/$/\t${p}/"  user.txt  #在这使用双引号，因为调用变量了，在不同行的用户名后面添加密码
	read name pass << EOF
	$(sed "${i}p"  user.txt)
EOF
	useradd $name
	echo $pass | passwd --stdin $name
	sed -i "${i}s/$/\t已创建/" user.txt
done
column -t user.txt
```

### 数据备份

1.备份/var/www/html 里面除了.tmp类型的所有文件到/opt/backup_data
2.备份的文件名要带时间戳，打tar包，格式为web_file_年-月-日.tar.gz
3.如果/opt/backup_data中备份的tar包凑齐5个之后，就都上传到目标服务器的/backup目录中并删除本地的这些tar包
4.任务执行成功或失败都要给出提示信息

```shell
vim backup.sh
#!/bin/bash
sou_path=/var/www/html
tar_path=/opt/backup_data
date=$(date +%F)
ex_file=*.tmp				#排除文件的格式
dest_ser_ip=192.168.88.90  #上传的目标服务器
mkdir -p ${tar_path}
tar -zcf ${tar_path}/web_file_${date}.tar.gz --exclude=$ex_file ${sou_path}
file_total=$(ls ${tar_path} | wc -l )
echo "${date}的文件已打tar包放入${tar_path}，目前备份的文件总数是${file_total}个"
if [ $file_total -ge 5 ];then
	scp ${tar_path}/* root@dest_ser_ip:/backup &> /dev/null
	if [ $? -ne 0 ];then
		echo "上传出错"
	else
		rm -rf ${tar_path}/web_file*
		echo "上传成功！"
	fi
fi
```

# Nginx服务配置

## 构建Nginx服务器

环境准备

| 主机名 | IP地址                                      |   角色    |
| :----: | :------------------------------------------ | :-------: |
| client | eth0：192.168.88.10/24                      |  客户端   |
| proxy  | eth0：192.168.88.5/24 eth1：192.168.99.5/24 | web服务器 |
|  web1  | eth1：192.168.99.100/24                     |     /     |
|  web2  | eth1：192.168.99.200/24                     |     /     |

### 源码安装Nginx

```shell
#tar解包
[root@proxy nginx-1.22.1] ./configure   --prefix=/usr/local/nginx --user=nginx --group=nginx --with-http_ssl_module  #指定安装路径，指定用户，指定组，开启SSL加密功能
[root@proxy nginx-1.22.1]~ make            #编译
[root@proxy nginx-1.22.1]~ make install    #安装
[root@proxy nginx-1.22.1]~ cd /usr/local/nginx/
[root@proxy nginx] ls
conf  html  logs  sbin
```

目录说明：
    conf 配置文件目录
    sbin 主程序目录
    html 网站页面目录
    logs 日志目录
**后续若是想要 ==添加模块==，但是不想删除之前的Nginx数据，可以执行之前的配置操作（./configure）参数与之前保持一样（通过主程序路径 -V 可以查看配置参数），再添加要安装的模块参数命令，make编译之后将objs目录下的nginx拷贝到nginx的sbin目录下代替现有的主程序**

```shell
[root@proxy nginx] killall  nginx
[root@proxy nginx] cd /root/lnmp_soft/nginx-1.22.1/
[root@proxy nginx-1.22.1] ./configure --with-stream --with-http_stub_status_module         #--with-stream开启4层代理模块，--with-http_stub_status_module开启status状态页面
[root@proxy nginx-1.22.1] make     #编译，不用执行make install安装，之间已经安装过
[root@proxy nginx-1.22.1] cp  objs/nginx /usr/local/nginx/sbin/ #覆盖原文件
cp: overwrite '/usr/local/nginx/sbin/nginx'? y   
[root@proxy nginx-1.22.1] /usr/local/nginx/sbin/nginx  -V
nginx version: nginx/1.22.1
built by gcc 8.5.0 20210514 (Red Hat 8.5.0-10) (GCC) 
configure arguments: --with-stream --with-http_stub_status_module
```

### 启动Nginx

```shell
useradd nginx -s /sbin/nologin   		#由于之前安装指定nginx程序的用户，所有需要创建对应的用户
/usr/local/nginx/sbin/nginx   #主程序路径启动服务
nginx服务默认通过80端口监听客户端请求
ss -antlp | grep 80
tcp   LISTEN 0      128          0.0.0.0:80          0.0.0.0:*     users:(("nginx",pid=7681,fd=6),("nginx",pid=7680,fd=6))
#其他命令
/usr/local/nginx/sbin/nginx -V 		#查看软件信息
/usr/local/nginx/sbin/nginx  -s reload  	#重新加载配置文件
/usr/local/nginx/sbin/nginx  -s stop 		#关闭服务
```

ss命令可以查看系统中启动的端口信息，该命令常用选项如下：
-a显示所有端口的信息
-n以数字格式显示端口号
-t显示TCP连接的端口
-u显示UDP连接的端口
-l显示服务正在监听的端口信息，如httpd启动后，会一直监听80端口
-p显示监听端口的服务名称是什么（也就是程序名称）

Nginx服务默认首页文件存储目录为/usr/local/nginx/html

## 虚拟主机

### 基于域名的虚拟主机

实验要求

配置基于域名的虚拟主机，实现两个基于域名的虚拟主机，域名分别为www.a.com和www.b.com

修改Nginx配置

```shell
cp conf/nginx.conf.default conf/nginx.conf  	#还原配置文件，前一个为默认配置备份
vim conf/nginx.conf
...
http {
... ...
	server {
		listen		80;				#端口
		server_name www.b.com;		#定义虚拟主机的域名
		location / {
			root	html_b;			#指定网页根路径
			index 	index.html index.htm;	#默认页面
		}
	}
	server {
	listen 		80;
	server_name		www.a.com;
	...
	location / {
		root html;
		index	index.html index.htm;
	}
	}
}
....
/usr/local/nginx/sbin/nginx
```

创建网站文件

```shell
echo "nginx-A~~~" > html/index.html
mkdir html_b
echo "nginx-B~~~" > html_b/index.html

client客户端测试
vim /etc/hosts			#修改hosts文件添加ip和域名的映射关系
192.168.88.5  www.a.com www.b.com

```

扩展补充：
windows环境配置hosts文件
C:\Windows\System32\drivers\etc\hosts
然后用文本打开hosts，在最后添加
192.168.88.5 www.a.com www.b.com

如果hosts文件是只读，可以
右击hosts文件---属性---安全---编辑---users---完全控制打钩

### 其他类型虚拟主机

#### 基于端口的虚拟主机

```shell
[root@proxy nginx] vim conf/nginx.conf  
...
    server {
        listen       8080;               #端口
        server_name  www.a.com;          #域名
        ......
}
    server {
        listen       8000;                #端口
        server_name  www.a.com;           #域名
      .......
}
...
[root@proxy nginx] sbin/nginx  -s  reload   #重新加载配置文件

client客户端测试
[root@client ~] curl  www.a.com:8080
nginx-B~~~
[root@client ~] curl  www.a.com:8000
nginx-A~~~
```
#### 基于IP的虚拟主机

```shell
[root@proxy nginx] vim conf/nginx.conf  
...
   server {
        listen       192.168.88.5:80;    #IP地址与端口
        server_name  www.a.com;          #域名
  ... ...
}
    server {
        listen       192.168.99.5:80;     #IP地址与端口
        server_name  www.a.com;
... ...
}
...
[root@proxy nginx] /usr/local/nginx/sbin/nginx -s reload   #重新加载配置文件
```

## SSL虚拟主机

配置基于加密网站的虚拟主机，实现一下目标：

​	1.该站点通过https访问

​	2.通过私钥、证书对该站点所有数据加密

### 加密算法说明

源码安装Nginx时必须使用--with-http_ssl_module参数，启用加密模块，对于需要进行SSL加密的站点添加SSl相关指令（设置网站需要的私钥和证书）

加密算法一般分为对称算法、非对称算法、信息摘要
对称算法有：AES、DES，主要应用在单机数据加密
非对称算法有：RSA、DSA，主要应用在网络数据加密
信息摘要：MD5、sha256，主要应用在数据完整性校验
```shell
[root@proxy nginx] echo 123 > /root/test
[root@proxy nginx] md5sum  /root/test

[root@proxy nginx] echo 1234 > /root/test  #更改文件内容
[root@proxy nginx] md5sum  /root/test      #md5值已经发生变化
```
### 配置SSL虚拟主机
1. 修改Nginx配置
```shell
[root@proxy nginx] cp conf/nginx.conf.default conf/nginx.conf  #还原配置文件
[root@proxy nginx] vim  /usr/local/nginx/conf/nginx.conf
...
#该位置再默认配置中的注释部分中
server {
        listen       443 ssl;    #监听端口443，ssl使用安全加密技术
        server_name            localhost;
        ssl_certificate      cert.pem;            #这里是证书文件
        ssl_certificate_key  cert.key;            #这里是私钥文件
        ssl_session_cache    shared:SSL:1m;
        ssl_session_timeout  5m;
        ssl_ciphers  HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers  on;
        location / {
            root   https;                        #加密网站根目录，更改，也可以自行定义
            index  index.html index.htm;
        }
    }
...    
```
2. 生成私钥和证书
```shell
[root@proxy nginx] openssl genrsa > conf/cert.key            #生成私钥，放到cert.key文件
[root@proxy nginx] openssl req -x509 -key conf/cert.key > conf/cert.pem    #-x509格式，生成证书,生成过程会询问诸如你在哪个国家之类的问题，可以随意回答
Country Name (2 letter code) [XX]:dc        #国家名
State or Province Name (full name) []:dc    #省份
Locality Name (eg, city) [Default City]:dc  #城市
Organization Name (eg, company) [Default Company Ltd]:dc    #公司
Organizational Unit Name (eg, section) []:dc               #部门
Common Name (eg, your name or your server's hostname) []:dc #服务器名称
Email Address []:dc@dc.com          #电子邮件

[root@proxy nginx] /usr/local/nginx/sbin/nginx -s reload   #重新加载配置文件
[root@proxy nginx] mkdir https    #创建安全网站的目录
[root@proxy nginx] echo "nginx-https~"  > https/index.html     #创建安全网站的页面
```

3. 客户端验证
```shell
命令行测试
[root@client ~] curl  -k  https://192.168.88.5     #检验，-k是忽略安全风险
nginx-https~      #看到这个内容就说明实验成功

真机浏览器访问：https://192.168.88.5
```

## LNMP环境

环境说明

|      主机名       | IP地址                  |   角色    |
| :---------------: | :---------------------- | :-------: |
| server1（已存在） | eth0：192.168.88.254/24 |  客户端   |
|  proxy（已存在）  | eth0：192.168.88.5/24   | web服务器 |

动态网站说明
静态网站：在不同环境下访问，网站内容不会变化
动态网站：在不同环境下访问，网站内容有可能发生变化

目前的网站一般都会有动态和静态数据，默认nginx仅可以处理静态数据，用户访问任何数据都是直接返回对应的文件，如果如果访问的是一个脚本的话，就会导致直接返回一个脚本给用户，而用户没有脚本解释器，也看不懂脚本源代码，因此需要整合LNMP（Linux、Nginx、MYSQL、PHP）实现动态网站效果

软件安装，MariaDB，php，php-fpm

```shell
[root@proxy nginx-1.22.1] yum -y install mariadb  mariadb-server mariadb-devel php php-mysqlnd php-fpm

#mariadb（数据库客户端软件）、mariadb-server（数据库服务器软件）、mariadb-devel（依赖包）、php（识别php语言）、php-fpm（进程管理器服务）、php-mysqlnd（PHP的数据库扩展包）
```

### 启动服务

```shell
1）启动Nginx服务
[root@proxy nginx-1.22.1] /usr/local/nginx/sbin/nginx
[root@proxy nginx-1.22.1] ss -antlp | grep 80
tcp   LISTEN 0      128          0.0.0.0:80        0.0.0.0:*    users:(("nginx",pid=15507,fd=6),("nginx",pid=15506,fd=6))

2）启动MySQL服务
[root@proxy nginx-1.22.1] systemctl enable --now mariadb  #加入开机自启并立即启动
[root@proxy nginx-1.22.1] systemctl status mariadb  #查看服务状态

3）启动PHP-FPM服务
[root@proxy nginx-1.22.1] systemctl enable --now php-fpm  #加入开机自启并立即启动
[root@proxy nginx-1.22.1] systemctl status php-fpm          #查看服务状态

4）使用PHP测试页面
[root@proxy nginx-1.22.1] cp /root/lnmp_soft/php_scripts/test.php /usr/local/nginx/html/   #拷贝动态网站测试页面到nginx中
使用浏览器访问192.168.88.5/test.php 则无法看到页面内容，而是会当成要下载的文件，因为无法解析php动态页面
```

### 配置动静分离

#### 使用IP端口方式连接

通过调整Nginx服务端配置，实现PHP网页解析，通过配置Fast-CGI支持
Fast-CGI是快速公共（通用）网关接口，可以连接如nginx等网站程序到网站的语言解释器(比如php) ，php-fpm进程使用了Fast-CGI解析动态网站页面
修改Nginx配置
```shell
vim /usr/local/nginx/conf/nginx.conf
...
65		location ~ \.php$ {				#~是使用正则表达式匹配以.php结尾的
66			root 		html;			
67			fastcgi_pass 127.0.0.1:9000;		#将请求转发给本机php-fpm的9000端口
			fastcgi_index	index.php;		#网站默认页
			include			fastcgi.conf;		#加载fastcgi配置文件
}
```
修改php-fpm配置文件
```shell
vim /etc/php-fpm.d/www.conf
...
38  listen = 127.0.0.1:9000  		#更改php-fpm端口号

systemctl restart php-fpm
ss -antlp | grep 9000
LISTEN 0      128        127.0.0.1:9000      0.0.0.0:*    users:(("php-fpm",pid=15808,fd=8),("php-fpm",pid=15807,fd=8),("php-fpm",pid=15806,fd=8),("php-fpm",pid=15805,fd=8),("php-fpm",pid=15804,fd=8),("php-fpm",pid=15803,fd=6))

115 pm.max_children = 50        #最大进程数量
120 pm.start_servers = 5        #最小进程数量
```
测试
启动或者重新加载Nginx
将动态网站测试页面拷贝到nginx中,lnmp_soft/php_scripts/mysql.php
客户端浏览器访问192.168.88.5/mysql.php
修改数据库内容，输入mysql进入数据库，create user dc@localhost identified by '123';
浏览器再次访问可以看到新创建的用户
#### 使用socket方式连接
更改php-fpm配置
```shell
vim /etc/php-fpm/www.conf
38 listen = /run/php-fpm/www.sock 			#socket方式（使用进程通信）
55 listen.acl_users = apache,nginx,nobody        #添加nobody账户

修改Nginx配置
vim /usr/local/nginx/conf/nginx.conf
...
 65         location ~ \.php$ {     #匹配以.php结尾
 66             root           html;
 67             fastcgi_pass   unix:/run/php-fpm/www.sock;  #将请求转发给php-fpm进程
 68             fastcgi_index  index.php;
 69             include        fastcgi.conf;        #加载fastcgi配置文件
 70         }
```
重新加载，浏览器可以访问192.168.88.5/test.php 可以看到页面内容

## 地址重写

关于Nginx服务器的地址重写，主要用到的配置参数是rewrite

语法格式：
rewrite regex replacement flag
rewrite 旧地址   新地址    [选项]

```shell
1）修改Nginx服务配置
#先还原配置文件
vim /usr/local/nginx/conf/nginx.conf
.. ..
server {
        listen       80;
        server_name  localhost;
        rewrite  /a.html  /b.html;       #新添加地址重写，a.html重定向到b.html  
        ...
    location / {
        root   html;
        index  index.html index.htm;
    }
}
echo "nginx-B~~" > /usr/local/nginx/html/b.html
#重新加载配置文件
#客户端测试
http://192.168.88.5/a.html          #内容显示的是nginx-B~~，但是地址栏没有发生变化，还是a.html页面
```

此时配置文件中直接写rewrite  /a.html  /b.html; 配置，在测试是其实会有些问题，比如在浏览器中访问时把192.168.88.5/a.html写成192.168.88.5/a.htmldc 或者写成 192.168.88.5/dc/a.html，访问都会正常显示b.html的页面，这是因为此时写的是<font color='blue'>只要包含a.html</font>的都会跳转，没有进行精准匹配，可以进行以下修改，只有写a.html时才会正确跳转

```shell
vim /usr/local/nginx/conf/nginx/conf
……
server {
	listen 		80;
	server_name	localhost;
	rewrite	  ^/a\.html$   /b.html;			
	
	....
}
```

参数

**redirect**  临时重定向，状态码302

**permanent** 永久重定向，状态码301

```shell
vim /usr/local/nginx/conf/nginx.conf
.. .. 
server {
	listen       80;
    server_name  localhost;
    rewrite ^/a\.html$  /b.html  redirect;      #新修改，redirect重定向，测试完之后把redirect换成permanent，是一样的效果
    ...
    location / {
    	root html;
    	index index.html index.htm;
    }
}
```

重新加载配置，浏览器访问http://192.168.88.5/a.html  #内容显示的是nginx-B~~，地址栏发生变化，是b.html页面

### 不同网站间的跳转

```shell
vim /usr/local/nginx/conf/nginx.conf
.. ..
server {
	listen 			80;
	server_name		localhost;
	rewrite /  http://www.bilibili.com/;        #新修改，访问旧网站的任意内容都跳转到新网站
    location / {
        root   html;
        index  index.html index.htm;
    }
}
```
重新加载配置，客户端浏览器访问http://192.168.88.5  
### 子页面重定向
修改配置文件(访问192.168.88.5/下面子页面，重定向至www.bilibili.com下相同的子页面)
```shell
vim /usr/local/nginx/conf/nginx.conf
.. ..
server {
	listen 			80;
	server_name		localhost;
	rewrite /(.*)  http://www.bilibili.com/$1;        #新修改
    location / {
        root   html;
        index  index.html index.htm;
    }
}
```
重新加载配置，客户端浏览器访问http://192.168.88.5  
### 不同浏览器跳转不同页面
```shell
vim /usr/local/nginx/conf/nginx.conf
.. ..
server {
	listen 			80;
	server_name		localhost;
	if ($http_user_agent ~* firefox) {   #如果用户使用了火狐浏览器
		rewrite /(.*) /firefox/$1;		#就进行地址重写，让用户看到火狐专用页面，否则就是其他页面；$http_user_agent是nginx的内置变量，包含了发起 HTTP 请求的客户端的用户代理（User-Agent）字符串，比如用的什么浏览器
	}
    location / {
        root   html;
        index  index.html index.htm;
    }
}
```
### 其他选项测试
last 不再读其他语句，但是会继续匹配其他location语句
break 不再读其他语句，结束请求
```shell
vim /usr/local/nginx/conf/nginx.conf
.. ..
server {
        listen       80;
        server_name  localhost;
        rewrite /a.html /b.html last;        #新修改
        rewrite /b.html /c.html;        #新修改
        ...
}
...
```

## Nginx反向代理

###  Nginx反向代理架构

![img](/home/student/Documents/v2-df48163acc6d2eb671f5f2fdd7ccd12d_720w.png)

虚拟环境

| 主机名            | IP地址                                         | 角色       |
| ----------------- | ---------------------------------------------- | ---------- |
| server1（已存在） | eth0：192.168.88.254/24                        | 客户端     |
| proxy（已存在）   | eth0:    192.168.88.5/24 eth1：192.168.99.5/24 | 代理服务器 |
| web1（已存在）    | eth1：192.168.99.100/24                        | web服务器  |
| web2（已存在）    | eth1：192.168.99.200/24                        | web服务器  |

web1,web2通过yum安装httpd服务实现简单的web服务，在初始页面写入不同的内容为之后的实验做区分。

还原Nginx配置，恢复到刚安装的默认配置，再修改配置文件/usr/local/nginx/conf/nginx.conf（源码安装的Nginx配置文件地址）

```shell
...
http {
...
#使用upstream定义后端服务器集群，集群名称任意(如webserver)
#使用server定义集群中的具体服务器和端口
	upstream webserver {
		server 192.168.99.100:80;
		server 192.168.99.100:80;
	}
	server {
		listen 		80;
		server_name localhost;
		...
		location / {
			root html;
			index index.html  index.htm;
			proxy_pass  http://webserver;   #通过proxy_pass将用户的请求发给webserver集群
		}
	}
}
```

重新加载Nginx配置后，使用客户机浏览器访问192.168.88.5，刷新可以看出网站的轮询效果，出现结构为web1或者web2提供的页面

#### 配置upstream集群池属性

设置权重：weight可以设置后台服务器的权重

```shell
...
	upstream webserver {
		server 192.168.99.100:80 weight=2;
		server 192.168.99.100:80;
	}
	server {
	.....
```

设置健康检查max_fails可以设置后台服务器连不上的失败次数，fail_timeout可以设置后台服务器的失败超时时间，等待多长时间再次尝试连接

```shell
...
	upstream webserver {
		server 192.168.99.100:80 weight=2;
		server 192.168.99.100:80 max_fails=2 fail_timeout=30;
	}
	server {
	.....
```

测试ip_hash，设置相同客户端访问相同web服务器

```shell
...
	upstream webserver {
		ip_hash;        #测试时客户端只会看见一个页面
		server 192.168.99.100:80 ;
		server 192.168.99.100:80 max_fails=2 fail_timeout=30;
	}
	server {
	.....
```

添加down标记，让集群主机暂时不参与集群活动

```shell
...
	upstream webserver {
		server 192.168.99.100:80 ;
		server 192.168.99.100:80 down; #web2服务器不参与集群访问
	}
	server {
	.....
```

## Nginx的TCP/UDP调度器（四层代理）

虚拟实验：后端SSH服务器两台，Nginx编译使用--with-stream 开启4层代理模块ngx_stream_core_module模块，Nginx采用轮询方式调用后端SSH服务器

虚拟配置同上

查询模块安装情况

```shell
/usr/local/nginx/sbin/nginx -V
nginx version: nginx/1.22.1
built by gcc 8.5.0 20210514 (Red Hat 8.5.0-10) (GCC) 
configure arguments: --with-stream
```

修改配置文件

```shell
vim /usr/local/nginx/conf/nginx.conf
....
	stream {		#配置写在http上面
		upstream backend {			#创建集群，名称backend
			server 192.168.99.100:22;	#后端SSH服务器IP和端口
			server 192.168.99.200:22;
		}
		server {		#调用集群
			listen 12345;		#Nginx代理监听的端口，可以自定义
			proxy_pass backend;		#调用backend集群
		}
	}
	http {
	....
	}

#启动Nginx服务，客户端访问代理服务器测试轮询效果
ssh 192.168.88.5 -p 12345
```

## Nginx常见问题

HTTP常见状态码列表：
<font color='blue'>200</font> 正常
<font color='blue'>301</font> & <font color='blue'>302</font> 重定向
<font color='blue'>400</font> 请求语法错误
<font color='blue'>401</font>  访问被拒绝
<font color='blue'>403</font>  禁止访问
<font color='blue'>404</font>  资源找不到
<font color='blue'>414</font>  请求URI头部太长
<font color='blue'>500</font>  服务器内部错误
<font color='blue'>502</font>  代理服务器无法正常获取下一个服务器正常的应答

### 查看nginx服务状态信息
编译安装时使用**--with-http_stub_status_module**开启状态页面模块
使用方式

```shell
vim /usr/local/nginx/conf/nginx.conf
....
	server {
		listen  80;
		server_name localhost;
		location /status {			#定义状态页面
			stub_status on;
			allow 192.168.88.5;   #允许88.5访问
			deny all;			#其他人拒绝访问
		}
	.....
	}
```
查看状态页面 192.168.88.5/status
参数说明
**Active connections**：当前活动的连接数量，有多少人访问网站
**Accepts**：已经接受客户端的连接总数量，有多少人曾经来过
**Handled**：已经处理客户端的连接总数量
**Requests**：客户端发送的请求数量
**Reading**：当前服务器正在读取客户端请求头的数量，请求头：客户现在正在发的请求，要看什么页面，要求服务器传过去
**Writing**：当前服务器正在写响应信息的数量，指服务器正在给客户回应信息
**Waiting**：当前多少客户端在等待服务器的响应，

### 优化Nginx并发量

使用ab高并发测试，命令` ab -n 1000 -c 100 IP `

-n任务量，总请求数

-c并发数，同时发送的请求数量（模拟的并发用户数）

测试分多轮完成（总请求数/并发数=轮次）

**Linxu系统默认限制文件打开数量1024**，<font color='blue'>`ulimit -n` </font> 查看最大文件数量，接数字临时修改最大文件数量，永久设置/etc/security/limits.conf

## uWSGI

基本概念：uWSGI是一个Web服务器，主要用途是将Web应用程序部署到生产环境中，可以用来连接Nginx服务与Python动态网站

环境准备

```bash
#安装python依赖软件
yum -y install gcc make python3 python3-devel
#安装python网站依赖包，peyton的软件包使用pip3安装（此处已经搜集的tar包和相关软件包，在自行安装）
[root@proxy python] pip3 install  pytz-2022.6-py2.py3-none-any.whl
[root@proxy python] pip3 install  Django-1.11.8-py2.py3-none-any.whl
[root@proxy python] pip3 install  django-bootstrap3-11.0.0.tar.gz
#可以自行寻找python网站代码测试
[root@proxy python] tar -xf python-project-demo.tar.gz #网站代码
[root@proxy python] cd python-project-demo/
[root@proxy python-project-demo] python3 manage.py runserver 0.0.0.0:8000      #manage.py相当于网站运行的引导文件
```

安装uWSGI

```bash
[root@proxy python]# pip3 install uWSGI-2.0.21.tar.gz
#准备配置文件（默认没有配置文件需要自己写）
[root@proxy python] vim myproject.ini
[uwsgi]
socket=127.0.0.1:8000			#与Web服务（Nginx）通信的端接口
chdir=/root/python/python-project-demo	#网站的工作目录
wsgi-file=learning_log/wsgi.py			#定义网站运行时，uwsgi调用的脚本文件
daemonize=/var/log/uwsgi.log

#运行uWSGI
[root@proxy python]# uwsgi --ini myproject.ini  #读取myproject.ini运行uWSGI
[uWSGI] getting INI configuration from myproject.ini
[root@proxy python]# ss -antlp | grep 8000  #8000已经监听，成功启动

#修改nginx配置文件，添加uWSGI转发
vim /usr/local/nginx/conf/nginx.conf 
....
	location / {
		uwsgi_pass 127.0.0.1:8000;		#动态页面交给uWSGI
		include uwsgi_params;			#加载调用uWSGI
		root  html;
		index index.html index.htm; 
	}
	location ~* \.(jpg|jpeg|gif|png|css|js|ico|xml)$ {
            expires 30d;			#设置缓存时间
         }
...

```

## 灰度发布

使用比较平稳的过度方式升级或者替换产品项目的方法统称

主要作用：

- 及时发现项目问题
- 尽早获取用户反馈信息，改进产品
- 若出现问题，将影响控制到最小范围

实现方式

基于IP的测试

创建用来测试的集群主机

```bash
upstream s8001 {			#测试的集群
	server 192.168.99.100:8001;
}
upstream default {			#原来的集群
	server 192.168.99.100;
	server 192.168.99.200;
}
```

设置Nginx变量，根据IP匹配对应的集群主机

```bash
set $group "default";
if($remote_addr ~ "192.168.99.1") {		#如果客户机ip是包含192.168.99.1就访问新版本业务集群1，$remote_addr为nginx的内置变量，代表访问者的ip
	set $group s8008;
}
```

基于用户ID的测试

通过修改网页文件去匹配用户对应的集群服务

<img src="/home/student/Documents/截图_选择区域_20250419100120.png" style="zoom:75%;" />

## 网站限速

limit_rate指令限制速度

```bash
http {
...
limit_rate 100k;			#全局限速
server {
	listen 80;
	server_name localhost;
	limit_rate 200k;		#虚拟主机限速
	...
	location /file_a {
		limit_rate 300k;	#用户访问file_a 目录下的内容限速300k
	}
	location /file_b {
		limit_rate 0k;		#不限速
	}
}
}
```

# RPM打包

制作Nginx的RPM包

安装rpm-build软件包：yum -y install rpm-build

生成rpmbuild目录结构

```bash
rpmbuild -ba nginx.spec 		#会报错，没有文件或目录
ls /root/rpmbuild
BUILD BUILDROOT  RPMS  SOURCES  SPECS  SRPMS   #SOURCES存放源码包的位置，RPMS存放制作好的rpm位置，SPECS存放制作软件包配置文件的位置
```

将源码包复制到SOURECS目录

创建并修改SPEC配置文件，文件名任意，已.spec结尾

```bash
vim /root/rpmbuild/SPECS/nginx.spec
Name:nginx                                 #源码包软件名称
Version:1.22.1                             #源码包软件的版本号
Release:1                                  #制作nginx的RPM包版本号
Summary:nginx is a web server              #RPM软件的概述
#Group:                                    #组，目前不需要，直接注释或者删除
License:GPL                                #License授权协议，GPL自由软件
URL:www.test.com                           #网址，自行定义
Source0:nginx-1.22.1.tar.gz                #源码包文件的全称

#BuildRequires:         #制作RPM时安装的软件包gcc，make，pcre-devel，openssl-devel等，即构建依赖，不用写，直接注释，在这只是提示作用，还是需要自己安装
Requires:pcre-devel openssl-devel       #rpm包制作好之后，安装RPM时的依赖关系
%description
nginx is a web server                   #软件的详细描述

%prep
%setup -q                               #自动解压源码包，并cd进入目录

%build
./configure                             #配置
make %{?_smp_mflags}

%install
%make_install

%files
%doc
/usr/local/nginx/*                      #对哪些文件与目录打包

%changelog
```

安装依赖软件包：gcc,make,pcre-devel,openssl-devel

rpmbuild创建RPM软件包：

```bash
rpmbuild -ba /root/rpmbuild/SPECS/nginx.spec
ls /root/rpmbuild/RPMS/x86_64/nginx-1.22.1-1.x86_64.rpm
yum -y install /root/rpmbuild/RPMS/x86_64/nginx-1.22.1-1.x86_64.rpm
```

# VPN服务器

概述：Virtual Private Network

- 在公用网络上建立专用私有网络，进行加密通讯
- 多用于为集团公司的各地子公司建立连接
- 连接完成后各个地区的子公司可以像局域网一样通讯

## WireGuard VPN

WireGuard是一种开源的VPN协议，旨在提供**高性能**、**低延迟**和**简单易用的安全通信**解决方案。

**工作原理是基于点对点的连接**，每个节点都有自己的公钥和私钥，实现安全的通信。

安装软件 `kmod-wireguard、 wireguard-tools`(客户端与服务端)，虚拟平台`Rocky Linux 8.6`

制作密钥文件（服务端）

```bash
wg genkey | tee private.key | wg pubkey > public.key
cat private.key
GB2NbtPoAEvNufEggKM41GNEUBlxfJfVYn4i9yJ4WlU=
cat public.key
UygBBCi6gEX5aJ0hMpKjBXDxltsV4+yI4NQTqK1ih1k=
#编写配置文件
vim /etc/wireguard/lwg.conf
    [Interface]		#服务端配置
    Privatekey =GB2NbtPoAEvNufEggKM41GNEUBlxfJfVYn4i9yJ4WlU=   #服务器的私钥
    Address = 10.10.10.1/8		#VPN内部使用的IP和掩码
    ListenPort = 54321			#wireguard服务监听的端口

    [Peer]  #对端（客户端）配置
    Publickey = I4PSki05VteWED39gh9XV03mBbkw/kCkq1QmQuT0XD0=   #客户端的公钥
    AllowedIPs = 10.10.10.2/32

#启动服务
wg-quick up lwg  #lwg是之前配置文件的名称
ss -antlpu | grep 54321 #检查端口，有54321
```

客户端配置

```bash
#制作密钥文件
vim /etc/wireguard/client.conf 
    [Interface]
    Privatekey = 6NQ2iV8v+9pcUSW8omqWEr3k2ITYNhrZUq7WuosZj3I=  #客户端生成的私钥
    Address = 10.10.10.2/8		#VPN客户端的IP和掩码
    DNS = 8.8.8.8  #VPN客户端的DNS

    [Peer] #对端，需要连接的服务器
    Publickey = UygBBCi6gEX5aJ0hMpKjBXDxltsV4+yI4NQTqK1ih1k=  #服务器生成的公钥
    Endpoint = 192.168.88.5:54321  #VPN服务器的公网IP地址和端口
    AllowedIPs = 10.0.0.0/8  #哪些流量经过VPN传递

[root@client wireguard] wg-quick up client  #启动VPN客户端
[root@client wireguard] ping 10.10.10.1  #测试可以连通服务端的VPN内部地址表示成功
```

# 内网穿透

**将内部私有网络（如家庭网络、企业网络）的服务暴露在公共网络，使外部用户可以访问内部网络中的服务**。通常用于远程访问、局域网共享、设备管理等场景，提供一种安全、便捷的远程访问解决方案。

**FRP（Fast Reverse Proxy）是一款开源的内网穿透工具**，旨在帮助用户轻松实现内外网之间的穿透，实现从外部访问内网的服务。

特点：**多种协议支持、安全、灵活配置、跨平台**

<img src="/home/student/Documents/截图_选择区域_20250419164011.png" style="zoom:75%;" />

**实验**

任意设备想要访问一台内网Linux客户机的远程服务(sshd)

**配置服务端**

```bash
vim frps.toml  	#服务端配置文件
bindPort = 7000  		#确定服务端的端口号，内网机器连接的端口
./frps  & 		#执行服务端程序
```

**客户端**

```
mv frpc.toml  frpc.ini 		#主程序识别的文件为frpc.ini，要改个名字不然识别不到
vim frpc.ini  
    serverAddr = "192.168.88.5"  #服务端IP
    serverPort = 7000		#服务端的端口号

    [[proxies]]		#设置代理信息
    name = "test-tcp"		#名称，随意
    type = "tcp" 		#协议
    localIP = 22	#本地业务端口
    remotePort = 6000  #服务端开启的代理端口，其他主机通过服务端的6000端口即可连接客户端的22号端口

./frpc &  #开启客户端程序
```

在服务端查看结果测试

```bash
ss -antlp | grep frps		#可以看到服务端自动开启6000端口
```

