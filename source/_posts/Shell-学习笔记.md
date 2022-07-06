---
title: Shell 学习笔记
categories: 分享
keywords: Shell
top_img: /img/5079067.webp
cover: /img/1776164.webp
abbrlink: aa327429
date: 2022-07-06 23:26:56
---

##  Shell 学习笔记

慕课链接：https://coding.imooc.com/class/chapter/314.html

每次想直接执行脚本的时候，总存在某些地方不会写，然后查资料，就比较浪费时间

趁着复习周，给自己加餐下，找了个慕课的教程，以下就记录下自己觉得重点的内容

<img src="https://pic.hengy1.top/typora/202206111304062.png" alt="image-20220611125652945" style="zoom:33%;" />

##  变量的相关使用

```
$# 表示参数个数

$0 是脚本本身的名字

$1 是传递给该shell脚本的第一个参数

$2 是传递给该shell脚本的第二个参数

$@ 表示所有参数，并且所有参数都是独立的

$$ 是脚本运行的当前进程ID号

$? 是显示最后命令的退出状态，0表示没有错误，其他表示有错误
```

### 变量的删除/替换

<img src="https://pic.hengy1.top/typora/202206111316663.png" alt="image-20220611131123947" style="zoom:33%;" />

<img src="https://pic.hengy1.top/typora/202206111316185.png" alt="image-20220611131456444" style="zoom:33%;" />

可以看出来使用了匹配规则后`I lov` 匹配到了就被删除了，当然其他也是相同道理

### 字符串处理

> expr 是从1开始计算；${string: position}是从0开始计算
```
获得字符串的长度：${#string}

获取子串在字符串中索引的位置：\`expr index $string $substring`
```

<img src="https://pic.hengy1.top/typora/202207062343774.png" alt="image-20220611132910356" style="zoom:33%;" />

- 抽取子串：当使用负数的时候需要加个**空格**

<img src="https://pic.hengy1.top/typora/202206111324725.png" alt="image-20220611132413220" style="zoom:33%;" />

```shell
#!/bin/bash
#

function echo_len # 使用函数打印出string的长度
{
		echo "${#string}" # 可以加 “”
}
# 交互性
while true
do
	read -p "Please input your choice." choice
	case $choice in
			1)
					echo_len
					;;
			q)
					exit
					;;
			*)
					echo "error"
					;;
	esac
done
```

### 命令替换

方式有2个：也就是前面一直出现的 1. \`command` 2. $(command)

1. 获取该系统的所有用户   users=\`/cat /etc/passwd | cut -d ":" -f 1`
2. 根据系统时间计算今年   echo "This is $(date +%Y) year"
3. 如果存在计算的话 echo "$(($(date +%j)/7))" 【使用$(())包住】
4. 判断nginx进程是否存在

```shell
#!/bin/bash
# 
process_num = $(ps -ef | grep nginx | grep -v grep | wc -l)
if [ $process_num -eq 0 ];then
	systemctl start nginx
if
```

###  Expr数学运算

方式有2个：1. expr $num1 operator $num2 2. $(( $(num1) operator $(num2) ))

注意需要转译的情况以及空格的情况 例如 `expr $num1 /> $num2`

判断输入的是不是整数：expr $num1 + 1；echo $? 如果是0的话就是正确的没有报错

```shell
#!/bin/bash
#

read -p "Please input your num." num
expr $num + 1 &> /dev/null # 不希望输入到终端
if [ $? -eq 0 ];then
		if [ `expr $num \> 0` -eq 1 ]; then
				echo "Yes"
		else
				exit
		fi
else
		echo "Error"
		exit
fi
```

### Bc内建运算器

这里支持浮点数等运算： `/usr/bin/bc`进入交互式的运算

如果想使用命令行进行计算的话：将运算传入进去

1. `echo "23.3+35" | bc `
2. `echo "scale=4;23.3+35" | bc`

```shell
#!/bin/bash
#
read -p "Please input your num1." num1
read -p "Please input your num2." num2
echo "scale=4;$(num1)/$(num2)" | bc
num3=`echo "scale=4;$(num1)/$(num2)" | bc`
```

##  函数使用

###  定义与使用

使用` $1, $2, $3 `进行传参

```shell
#!/bin/bash
#

function calcu
{
	case $2 in
		+)
			echo "`expr $1 + $3`"
		-)
			echo "`expr $1 - $3`"
		\*)
			echo "`expr $1 \* $3`"
}
calcu $1 $2 $3
```

然后在终端输入`sh calculate.sh 20 + 20`是能够直接赋值进去的

### 函数的返回值

- 使用return返回值：只能返回1-255的整数，通常用来判断状态
- 使用echo返回值：可以返回任何字符串

```shell
#!/bin/bash
#

function is_nginx_running
{
    this_pid=$$
    ps -ef | grep nginx | grep -v grep | grep -v $this_pid &> /dev/null
    if [ $? -eq 0 ];then
      return 0
    else
     	return 1
}

is_nginx_running && echo "Nginx is running" || echo "Nginx is not running"
```

```shell
#!/bin/bash
#

function get_users
{
		users=`cat /etc/passwd | cut -d: -f1`
		echo $user
}

user_list=`get_users`
for u in $user_list
do 
	echo $u
done
```

### 局部变量与全局变量

不做特殊说明，Shell的变量都是默认变量

```shell
#!/bin/bash
#

var1="Hello world"
function test
{
		var2=87
		local var3=11
}

echo $var1
echo $var2
test
echo $var1
echo $var2  # 调用了就生效了
echo $var3  # 说好了是local
```

## 文件查找

`Find [路径] [选项] [操作]`

```bash
$ find /etc -name "*.conf" 

$ find /etc -iname "*.conf" # 忽略大小写

$ find . -type f # 搜索文件会往深度走

$ find . -size +1M # 搜索大于1M的文件

$ find . -mtime -3 # 搜索3天之内修改

$ find . -type f -mindepth 2 # 从当前的二级子目录开始查询

$ find . -type f -maxdepth 2 # 最多搜索到二级子目录

$ find . -perm 777 # 从当前查找777权限的

$ find ./etc -name "*.conf" -exec rm -rf {} \;  # 查找出来并执行动作

# -a yu
# -o 或
# ！非
```

- find 在遍历磁盘

- locate 在数据库进行查找
  - 使用updatedb去更新数据库
- whereis查找二进制文件在哪里
  - -b 查找可执行文件在哪里
- which 仅查找二进制程序文件

## 文本处理三剑客

###  grep和egrep

```bash
$ grep python_text file # 在file中查找python_text

$ grep -n python_text file # 看行号

$ grep -iv python_text file # 忽略并忽略大小写

$ grep -E "python_text|Python" file # 扩展表达式

$ grep -F "py.*" file # 不使用正则

$ grep -r "python" # 从父目录递归搜索
```

###  sed

> 流编辑器，对标准输出或文件进行处理

```bash
$ sed -n '/py.*/p' demo.txt  # 每一行去处理 -n 只打印匹配行

$ sed -n -e '/py.*/p' -e '/PY.*/p' demo.txt # 多个

$ sed -n -r '/python|PYTHON/' demo.txt # 支持拓展表达式

$ sed -n 's/love/like/g;p' demo.txt # 替换并打印出来但是没有修改原先的

$ sed -n -i 's/love/like/g;p' demo.txt # demo.txt被修改了
```

```bash
# pattern用法

# /pattern1/command 匹配pattern1行执行命令
# /pattern1/,/pattern2/command 匹配pattern1的行开始到匹配到pattern2的行结束
# 10,/pattern1/command 匹配到第10行开始到pattern1结束
# /pattern1/,10command 匹配到pattern1的行开始到第10行结束

$ sed -n "17p" /etc/passwd # 打印第17行

$ sed -n "17,20p" /etc/passwd # 打印第17到20行

$ sed -n "/bash/p" /etc/passwd # 匹配pattern1行执行命令
```

```bash
# 编辑命令
# p 打印
# a 行后追加 # i 行前追加
# r 外部读入后行后追加 # w 匹配行写入外部
# d 删除
# s/old/new 第一个替换
# s/old/new/g 行内全部替换
# s/old/new/2g 行内前2个替换
# s/old/new/ig 全部替换忽略大小写

$ sed -i '/\/bin\/bash/a This is User!' ./demo.txt # 能匹配到的行在这个行后追加新的一行

$ sed -n '/\/bin\/bash/w /tmp/user_login.txt' /etc/passwd # 匹配到的行到/tmp/user_login.txt

$ sed -i 's/\/bin\/bash/\/BIN\/BASH/' /etc/passwd # 如果没有g的话只会替换匹配到当前行的第一个

$ sed -i 's/\/bin\/bash/\/BIN\/BASH/g' /etc/passwd # 全局替换

$ sed -n '/\/bin\/bash/n' /etc/passwd # 显示行号

$ sed -i 's/had...p/hadoops/g' demo.txt # 任意匹配3个

$ sed -i 's/had..p/&s/g' demo.txt # &实现了反向引用

$ sed -i 's/\(had..p\)/\1O/g' demo.txt # \1实现了反向引用 更加灵活 需要\(\)

$ sed -i 's/\(had\).../\1doop/g' demo.txt
```

```shell
#!/bin/bash
#
# 脚本写法

old_str=hadoop
new_str=HADOOP

sed -i "s/$old_str/$new_str/g" demo.txt # 当作变量的话使用双引号
```

```bash
# 删除操作
$ sed -i '15d' /etc/passwd # 删除第15行

$ sed -i '8,14d' /etc/passwd # 删除8-14行

$ sed -i '/\/sbin\/nologin/d' /etc/passwd

$ sed -i '/^mail/,/^yarn/d' /etc/passwd

$ sed -i '/\/sbin\/nologin/, 13d' /etc/passwd

$ sed -i '5, /^ftp/d' /etc/passwd

# 删除所有的注释行与空行
$ sed -i '/^$/d;/[:blank:]*#/d' nginx.conf

# 在不是#开头加*号
$ sed -i '/^[^#]/\*&/g' nginx.conf
```

```bash
# 修改操作
$ sed -i '1s/root/ROOT' passwd # 第一行中第一个

$ sed -i '5,10s/\/sbin\/nologin\/bin\/bash/g' # 5-10行所有

$ sed -i '/\sbin\/nologin/s/login/LOGIN/g' # 匹配到/sbin/nologin的行中把login换大写

$ sed -i '/^root/, 15s/nologin/SPARK/g' passwd # 匹配到root开头到15行替换

$ sed -i 's/[0-9]*//g' file.txt # 删除数字
```

```bash
# 追加内容
$ sed -i '10a Add Line' /etc/passwd # 第10行下面加一行
$ sed -i '10,20a Add Line' /etc/passwd # 10-20每一行下面加一行
$ sed -i '/\/bin\/bash/a Insert Line' /etc/passwd # 匹配到/bin/bash加

$ sed -i '/^yarn/i Add Line before' /etc/passwd
$ sed -i 'i Line before' /etc/passwd # 每一行前面

$ sed -i '20r /etc/fstab' /etc/passwd # 20行后面
$ sed -i '\/bin\/bash/r /etc/fstab' /etc/passwd # 匹配到的加

$ sed -i '\/bin\/bash/w /tmp/demo.txt' /etc/passwd 
$ sed -i '10,/^hdfs/w /tmp/demo.txt' /etc/passwd 
```

###  awk

> awk 'BEGIN{}pattern{commands}END{}' file
>
> - BEGIN{} 正式处理之前先执行
> - pattern 匹配模式
> - {commands} 处理命令，可能多行
> - END{} 处理完所有匹配数据后执行

```bash
# awk中的常用选项
# -v 参数传递
# -f 指定脚本文件
# -F 指定分隔符号
# -V 查看awk版本号

$ awk -v num2="$num2" 'BEGIN{print num2}' # 外部的变量引入awk
$ awk -F ":" '{print $1}' # 可以使用F来指定分隔符
```

```bash
# $0 整行内容
# $1-n 为当前行的第几个（可以自己切）
# NF 当前行的字段个数
# NR 当前行的行号
# FNR 从0开始，多个文件
# FS 输入字段分隔符 不指定为空格或者Tab
# RS 输入行分隔符 默认为回车
# OFS 输入字段分隔符 默认为空格
# ORS 输出行分隔符
# FILENAME 当前输入文件名字
# ARGC 命令行参数个数
# ARGV 命令行参数数组

$ awk '{print $0}' /etc/passwd # 输出全部
$ awk 'BEGIN{FS=":"}{print $1}' /etc/passwd
$ awk '{print NF}' /etc/passwd # 查看分隔后每行的字段个数
$ awk 'BEGIN{RS="--";FS="|";ORS="&"}{print $2}' /etc/passwd # 自己分行自己切
$ awk 'BEGIN{RS="--";FS="|";ORS="&";OFS=":"}{print $2,$3}' /etc/passwd
$ awk '{print $NF}' /etc/passwd # 可以作为变量
```

```bash
# printf 格式化输出
# s 字符串 # d 十进制
# f 浮点数 # x 十六进制 # o 八进制
# e 科学技术码 # c 单个字符的ASCII

$ awk '{printf "%s\n", $1}' /etc/passwd
$ awk '{printf "%s %s\n", $1, $7}' /etc/passwd
$ awk '{printf "10%s 10%s\n", $1, $7}' /etc/passwd
$ awk '{printf "%-20s %-10s\n", $1, $7}' /etc/passwd # 左对齐
$ awk '{printf "%#x\n", $1}' /etc/passwd # 十六进制
```

```bash
# 模式匹配
# 1. 正则匹配
# 2. 关系表达
$ awk 'BEGIN{FS=":"}/root/{print $0}' /etc/passwd
$ awk 'BEGIN{FS=":"}/^yarn/{print $0}' /etc/passwd

$ awk 'BEGIN{FS=":"}$3<50{print $0}' /etc/passwd
$ awk 'BEGIN{FS=":"}$7=="/bin/bash"{print $0}' /etc/passwd
$ awk 'BEGIN{FS=":"}$3~/[0-9]{3,}/{print $0}' /etc/passwd # 第三个能匹配到3个数字 ～ 为匹配符号
$ awk 'BEGIN{FS=":"}$1=="root" || $1=="yarn" {print $0}' /etc/passwd
```

```bash
# 动作操作
$ awk 'BEGIN{var=20+10;var1="hello";print var}'
$ awk '/^$/{sum++}END{print sum}' /etc/passwd # 计算空白行
$ awk '{total=$2+$3+$4;AVG=total/3;printf "%-8s%5d%5d%5d%5f\n", $1,$2,$3,$4,AVG}' student.txt

# 条件语句
$ awk 'BEGIN{FS=":"}{if ($3 < 100 && $3 > 50) print $0}' /etc/passwd
$ awk 'BEGIN{FS=":"}{if ($3 < 100 && $3 > 50) {print $0} else {print $0}}' /etc/passwd

# 循环
$ awk 'BEGIN{while(i < 100){sum+=i;i++}print sum}' /etc/passwd
```

```bash
# 字符串内置函数
# length(str)
# index(str1, str2)
# tolower(str)
# toupper(str)
# substr(str, m, n)
# split(str, arr , fs)
# sub(RE, RepStr, str)
# gsub(RE, RepStr, str)

$ awk '{print length($1)}' /etc/passwd
```

数组的使用暂时用不上，就先不做笔记了