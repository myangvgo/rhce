# Chapter 01 编写 bash 脚本

[返回](../README.md)

[TOC]

## 1. 元字符和正则表达式

Shell 接收用户输入的命令后，根据空格将用户的输入拆分成 token，并扩展 token 里面的特殊字符，然后再调用相应的命令。

Shell 对特殊字符的扩展称为模式扩展（globbing），有些用到了通配符，称为通配符扩展。

### 1.1 通配符

通配符一般用于 shell

#### 通配符元字符

- `.` 通常没有意义

- `[]` 匹配一个字符
  - `[abc]` 匹配 a, b, c 中任意一个字符
  - `[a-z]` 匹配 a 到 z 中任意一个字符
  - `[A-Z]` 匹配 A 到 Z 中任意一个字符
  - `[0-9]` 匹配数字 0 - 9
  - `[a\-z]` 匹配 a, -, z 
  - `[!a-z]` 或者 `[^a-z]` 非 a - z 的任意一个字符

- `?` 表示任意一个字符
- `*` 表示任意多个字符
- `[[:alnum:]]`：匹配任意英文字母与数字
- `[[:alpha:]]`：匹配任意英文字母
- `[[:blank:]]`：空格和 Tab 键。
- `[[:cntrl:]]`：ASCII 码 0-31 的不可打印字符。
- `[[:digit:]]`：匹配任意数字 0-9。
  - 字符类的第一个方括号后面，可以加上感叹号`!`，表示否定
  - `echo [![:digit:]]*` 非数字
- `[[:graph:]]`：A-Z、a-z、0-9 和标点符号。
- `[[:lower:]]`：匹配任意小写字母 a-z。
- `[[:print:]]`：ASCII 码 32-127 的可打印字符。
- `[[:punct:]]`：标点符号（除了 A-Z、a-z、0-9 的可打印字符）。
- `[[:space:]]`：空格、Tab、LF（10）、VT（11）、FF（12）、CR（13）。
- `[[:upper:]]`：匹配任意大写字母 A-Z。
- `[[:xdigit:]]`：16进制字符（A-F、a-f、0-9）

```sh
# 正确的写法，避免了同名文件影响
yum list vsftpd\* -y

# yum是bash的子进程，如果当前目录里有vsftpdxx，则实际执行的是yum list vsftpdxx
yum list vsftpd*
```

### 1.2 正则表达式

正则表达式提供了一种便于查找特定内容的模式匹配机制。

#### grep 和 egrep

`grep` **g**lobal search **r**egular **e**xpression and **p**rint out the line，用于全面搜索的正则表达式，并将结果输出。

`egrep` 为扩展搜索命令，相当于 `grep -E`，支持扩展的正则表达式

```sh
grep keyword file

grep regexp file

# -P, --per-regexp, Perl regular expression
grep -P file

# -o, --only-matching, 只显示匹配 pattern 部分的行
grep -o
```

#### 正则表达式元字符

`^` 匹配行首

`$` 匹配行尾

`.` 通常表示任意一个字符

`?` 前面的字符出现了**0**次或者**1**次，此时可能需要使用扩展正则表达式

`+` 前面的字符出现了至少**1**次

`*` 前面的字符出现了**0**次或者多次

`{m}` 前面的字符出现了 m 次

`{m,}` 前面的字符出现了至少 m 次

`{,n}` 前面的字符最多出现 n 次

`{m,n}` 前面的字符至少出现了 m 次，但是不超过 n 次

`\b` 匹配词语两侧的空字符串

`\B` 匹配词语中间的空字符串

`\<` 匹配词语开头的空字符串

`\>` 匹配词语结尾的空字符串

`\w` 匹配词语部分

`\W` 匹配非词语部分

`\s` 匹配空格

`\S` 匹配非空格

建议使用单引号括起正则表达式，确保字符由正则表达式解释，而不是由 shell 解释

```sh

# 包含以单词 tom 开头的行
grep '\btom' file
grep '\<tom' file

# 包含以单词 tom 结尾的行
grep 'tom\b' file
grep 'tom\>' file

# 包含以 tom 开头和结尾完整的单词行
grep '\btom\b' file
grep '\<tom\>' file


# selinux port
semanage port -l | grep "80"
semanage port -l | grep "\b80\b"
[root@server133 ~]# semanage port -l | grep "\b80\b"
http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000

# tom 后面跟了0个或1个字符的行
egrep 'tom.?' file
```

#### 零宽断言（zero-width assertion）

> [正则表达式的先行断言(lookahead)和后行断言(lookbehind) | 菜鸟教程 (runoob.com)](https://www.runoob.com/w3cnote/reg-lookahead-lookbehind.html)

在使用正则表达式的时候，有时候需要捕获的内容前后必须是特定内容，但是又不捕获这些特定内容时，需要用到零宽断言。

零宽断言匹配到的内容不会保存到匹配结果中去，最终只是匹配一个位置。**它的作用是给指定位置添加一个限定条件**。

零宽断言一共有四种形式：

##### 正向先行断言 positive lookahead assertion

`(?=pattern)` 

代表字符串中的一个位置，紧接该位置之后的字符序列能够匹配 pattern。

```sh
# 只匹配 regular 中的 re
[root@server133 ~]# echo 'a regular expression' | grep -P 're(?=gular)'
a regular expression
```

##### 负向先行断言 negative lookahead assertion

`(?!pattern)` 

代表字符串中的一个位置，紧接该位置之后的字符序列不能匹配 pattern。

```sh
# 匹配除 regex, regular 之外的 re
[root@server133 ~]# echo 'regex represents regular expression' | grep -P 're(?!g)'
regex represents regular expression
```

##### 正向后行断言 positive lookbehind assertion

`(?<=pattern)` 

代表字符串中的一个位置，紧接该位置之前的字符序列能够匹配 pattern。

```sh
# 要想匹配单词内部的 re，但不匹配单词开头的 re;单词内部的 re，在 re 前面应该是一个单词字符
[root@server133 ~]# echo 'regex represents regular expression' | grep -P '(?<=\w)re'
regex represents regular expression
```

##### 负向先行断言 negative lookbehind assertion

`(?<!pattern)` 

代表字符串中的一个位置，紧接该位置之前的字符序列不能匹配 pattern。

```sh
# 要想匹配单词开头的 re
[root@server133 ~]# echo 'regex represents regular expression' | grep -P '(?<!\w)re'
# (?<!\w)re 相当于 (?<=\b)
[root@server133 ~]# echo 'regex represents regular expression' | grep -P '(?<=\b)re'
regex represents regular expression
```

## 2. 编写 bash 脚本

> [Bash 简介 - Bash 脚本教程 - 网道 (wangdoc.com)](https://wangdoc.com/bash/intro)

### 2.1 bash 脚本简介

#### 指定命令解释器

脚本的第一行通常以 `#!` 开头，通常称为 `sh-bang` 或者 `she-bang`，其 来源于字符 #（sharp）和字符 `!` （感叹号 bang）的名称。`#!` 后面的语法为处理该脚本的命令解释器。例如 Bash 脚本的开头为

```sh
#!/bin/bash
```

#### 执行 bash 脚本

完整的 bash 脚本必须为可执行文件，能作为常规命令运行。通过 `chmod` 添加可执行权限

#### 对特殊字符添加引号

如果需要使用特殊字符的字面值，通常需要转义，可以通过 `\`, 双引号，单引号来实现

**反斜杠、单引号**保留其所有字符的字面含义

```sh
[root@server133 ~]# echo \# not a comment
# not a comment
[root@server133 ~]# echo '# not a comment #'
# not a comment #
```

双引号可以阻止通配和 shell 扩展，但依然允许命令和变量替换。

```sh
[root@server133 ~]# var=$(hostname -s); echo $var
server133
```

### 2.2 变量

设置变量的方法：

```sh
# 1. 接受用户输入
[root@server133 ~]# read -p "please input username: " username
please input username: admin
[root@server133 ~]# echo $username
admin

# 2. 显式定义
[root@server133 ~]# usergroup=ad
[root@server133 ~]# echo $usergroup
ad

# 3. 将命令的结果赋值给变量
# 使用 `` 或者 $()
[root@server133 ~]# ip=`ifconfig ens160 | awk '/inet /{print $2}'`
[root@server133 ~]# echo $ip
192.168.26.133
[root@server133 ~]# ip=$(ifconfig ens160 | awk '/inet /{print $2}')
[root@server133 ~]# echo $ip
192.168.26.133
```

删除变量

```sh
unset varname

[root@server133 ~]# unset username
[root@server133 ~]# echo $username
```

#### 本地变量

只影响当前的 Shell，不能影响子 Shell

```sh
变量名=值
变量名="值1 值2"
```

* 等号两边不能有空格
* 值的部分如果有空格，需要使用引号
* 如果变量的值本身也是变量，可以使用`${!varname}`的语法，读取最终的值

```sh
[root@server133 ~]# currentuser=USER
[root@server133 ~]# echo ${!currentuser}
root
```

#### 环境变量

* 可以影响子 Shell，不能影响父 Shell
* 如果定义的变量要在所有终端都生效，需要在 `~/.bash_profile` 中定义
  * 登录过程：先执行用户家目录下的 `.bash_profile`
  * 然后执行用户家目录下的 `.bashrc`，一般用于定义 alias
  * 最后再执行 `/etc/bashrc`

```sh
export myenv=DEV
# 或者
myenv=DEV
export myenv

[root@server133 ~]# export myenv=DEV
[root@server133 ~]# echo $$
2115
[root@server133 ~]# bash
[root@server133 ~]# echo $$
2300
## 子 Shell 可以访问环境变量的值
[root@server133 ~]# echo $myenv
DEV
[root@server133 ~]#
```

* 系统自带的环境变量

```sh
# 列出系统自带的环境变量
[root@server133 ~]# env
[root@server133 ~]# printenv

[root@server133 ~]# echo $USER
root
[root@server133 ~]# echo $HOME
/root
[root@server133 ~]# echo $UID
0
[root@server133 ~]# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
[root@server133 ~]#
```

`set` 命令可以显示所有变量（包括环境变量和自定义变量），以及所有的 Bash 函数。

#### 位置变量

`$0` 脚本名称

`$1` 第一个变量

`$2` 第二个变量

...

`${10}` 第十个变量，超过9之后，需要使用大括号

`$#` 参数总个数

`$*` 列出所有的参数

#### 预定义变量

`$$`为当前 Shell 的进程 ID

`$?`为上一个命令的退出码，用来判断上一个命令是否执行成功

`$_`为上一个命令的最后一个参数

```sh
[root@server133 ~]# grep dictionary /usr/share/dict/words
antidictionary
benedictionary
dictionary
dictionary-proof
extradictionary
nondictionary
[root@server133 ~]# echo $_
/usr/share/dict/words
```

`$!`为最近一个后台执行的异步命令的进程 ID

```sh
$ firefox &
[1] 11064

$ echo $!
11064
```

`$-`为当前 Shell 的启动参数

`$#` 参数总个数

`$*` 列出所有的参数

#### 变量默认值

* 如果变量不存在，返回一个默认值；但是不改变变量值

```sh
${varname:-word}

[root@server133 ~]# echo $count

# count 变量不存在时返回 0
[root@server133 ~]# echo ${count:-0}
0
```

* 如果变量存在且不为空，则返回它的值，否则将它设为默认值，并且返回默认值

```sh
${varname:=word}

[root@server133 ~]# echo $count2

[root@server133 ~]# echo ${count2:=0}
0
[root@server133 ~]# echo $count2
0
```

* 如果变量名存在且不为空，则返回默认值，否则返回空值。它的目的是测试变量是否存在

```sh
${varname:+word}

[root@server133 ~]# echo ${count:+1}

[root@server133 ~]# echo ${count2:+1}
1
```

* 如果变量`varname`存在且不为空，则返回它的值，否则打印出`varname: message`，并中断脚本的执行。如果省略了`message`，则输出默认的信息“parameter null or not set.”。它的目的是防止变量未定义。

```sh
${varname:?message}

[root@server133 ~]# filename=${fname:?"filename missing."}
bash: fname: filename missing.
```

### 2.3 返回值

`$?` 记录命令的返回值，执行成功的话，返回值为0，否则结果为非0

```sh
$ unknowncmd
bash: unknowncmd: command not found
$ echo $?
127
```

逻辑上的否定，返回值也是非零

```sh
# 命令执行成功，但是逻辑上为否定
$ systemctl is-active vsftpd
inactive
$ echo $?
3

# 查看字符是否存在，不关心输出
$ grep -q root /etc/passwd
$ echo $?
0
```

### 2.4 数值运算

`$(())`

```sh
$ echo $((1+1))
2

$ echo $((1*2))
2

$ echo $((3**2))
9

$ echo $((3/2))
1
```

`$[]`

```sh
$ echo $[1+2]
3

$ echo $[2*2]
4

$ echo $[3**2]
9

$ echo $[3/2]
1
```

`expr` 只支持简单运算

```sh
$ expr 1 + 2
3

$ expr 3 - 2
1

$ expr 3 \* 2
6
```

`let`

```sh
$ let sum=1+2
$ echo $sum
3

# vs

$ sum=1+2
$ echo $sum
1+2
```

`declare -i` 定义整数变量

```sh
$ declare -i sum
$ sum=1+2
$ echo $sum
3
```

`bc` 数字计算 （Binary Calculator）用于进行高精度的数值计算

```sh
# 设置精确到小数点后两位
$ bc
scale=2
1/2
.50

# 使用非交互模式输出结果
$ echo "scale=2 ; 1/2" | bc
```

### 2.5 比较

#### 三种比较的语法

`[ 比较 ]` 等同于 `test 比较`

`[[ 比较 ]]`支持通配符和正则

#### 数值

- `-eq`
- `-gt`
- `-ge`
- `-lt`
- `-le`
- `-ne`

#### 字符

- `==` 后面跟的通配符比较
- `=~` 后面跟的正则表达式比较
- `>`
- `>=`
- `<`
- `<=`
- `!=`

```sh
$ [[ 'a' < 'b' ]]
$ echo $?
0

$ [[ 'a' > 'b' ]]
$ echo $?
1

$ os=linux

# ? 为字符
$ [ $os == linu? ]
$ echo $?
1

# ? 为通配符
$ [[ $os == linu? ]]
$ echo $?
0

$ numstr=3x4
$ [[ $numstr =~ ^[0-9]+$ ]]
$ echo $?
1

$ numstr=34
$ [[ $numstr =~ ^[0-9]+$ ]]
$ echo $?
0
```

#### 文件属性

- `[ -属性 /path/file ]`
- `r` 读权限
- `w` 写权限
- `x` 执行权限
- `f` 普通文件
- `d` 目录
- `L` 连接
- `b` 块设备文件
- `e` 是否存在

```sh
cadmin@pc-de-min:~$ [ -x /etc/passwd ]
cadmin@pc-de-min:~$ echo $?
1
cadmin@pc-de-min:~$ [ -d /etc/passwd ]
cadmin@pc-de-min:~$ echo $?
1
cadmin@pc-de-min:~$ [ -f /etc/passwd ]
cadmin@pc-de-min:~$ echo $?
0
cadmin@pc-de-min:~$ [ -L /etc/passwd ]
cadmin@pc-de-min:~$ echo $?
1
```

### 2.6 判断语句

#### `if` 判断

```sh
if condition1 ; then
	cmd1
el if condition2 ; then
	cmd2
else
	cmdx
fi
```

`hello.sh`

```sh
#!/bin/bash

if [ $UID -ne 0 ] ; then
	echo "只有 root 用户才能打招呼"
	exit 1
fi

echo "Hello $USER"
```

`wc.sh` 统计文件行数

```sh
#!/bin/bash
if [ $# -eq 0 ] ; then
	echo "请输入文件名"
	exit 1
fi

if [ ! -f $1 ] ; then
	echo "$1 文件不存在"
	exit 1
fi

# 统计文件行数
wc -l $1
```

#### `case` 判断

```sh
case $变量 in
	val1)
		cmd1
		;;
	val2)
		cmd2
		;;
	*)
		cmdx
		;;
esac
```

`learn.sh`

```sh
#!/bin/bash

cat <<EOF
#################################
# Hello Bash                    #
#################################
EOF

# 等待用户输入
read -p "请问 Bash 命令能否正常执行(y/n)：" ans

case $ans in
	y|Y)
		echo "Bash 命令可正常执行"
	;;
	n|N)
		echo "请先配置好 Bash 命令"
		exit 1
	;;
	*)
		echo "只能输入 y/n"
	;;
esac

echo "Starting bash"
```

### 2.7 循环语句

#### `for` 循环

```sh
for 变量 in val1 val2 val3
do
	cmd1 $变量
done
```

```sh
# 打印 1 - 10
for i in 1 2 3 4 5 6 7 8 9 10
do
echo $i
done

# 或者
for i in $(seq 1 10) ; do
echo $i
done
```

批量创建用户 `init-users.sh`

```sh
#!/bin/bash

# 批量创建用户
for i in $(seq 1 5); do
	let u=2000+$i
	useradd -u $u user$i
	echo redhat | passwd --stdin user$i
done
```

批量删除用户 `del-users.sh`

```sh
#!/bin/bash

# 批量创建用户
for i in $(seq 1 5); do
	userdel -r user$i
done
```

#### `while` 循环

```sh
while condition
do
	cmd
done
```

打印 1 - 10

```sh
#!/bin/bash

declare -i n=1
while [ $n -le 10 ] ; do
	echo $n
	# let n = $n + 1
	n=$((n + 1))
done
```

read 读取文件一行，结合 while 可以读取整个文件的内容

```sh
# 读取文件，但是只打印文件中的一行
read usrpass < /etc/passwd
echo $usrpass

# 读取文件 ，逐行打印
while read usrpass ; do
echo $usrpass
done < /etc/passwd
```

写一个脚本 `count.sh`，实现 `wc -l` 统计文件行数的功能

```sh
#!/bin/bash
# 必须要带有一个参数
if [ $# -eq 0 ] ; then
	echo "必须要指定一个普通文件"
	exit 1
fi

# 该参数必须是一个存在的普通文件
if [ ! -f $1 ] ; then
	echo "$1 不存在或者不是一个普通文件"
	exit 1
fi

# 统计行数
declare -i n=0
while read line ; do
	let n=$n+1
done < $1

echo "$n $1"
```

```sh
cadmin@pc-de-min:~$ chmod +x count.sh
cadmin@pc-de-min:~$ ls -l count.sh
-rwxr-xr-x 1 cadmin cadmin 339 Oct 12 22:08 count.sh

cadmin@pc-de-min:~$ ./count.sh
必须要指定一个普通文件
cadmin@pc-de-min:~$ ./count.sh /etc
/etc 不存在或者不是一个普通文件
cadmin@pc-de-min:~$ ./count.sh /etc/hosts
17 /etc/hosts
```

### 2.8 函数

在脚本中，函数是可以重复执行的代码段。别名只适合封装简单的单个命令，函数则可以封装复杂的多行命令

```sh
function fn()
{
	# cmd
}

# or
fn()
{
	# cmd
}

# or 

function fn
{
	# cmd
}
```

`hello.sh`

```sh
#!/bin/bash

function hello() {
	echo "[$(date)] hello $USER"
}

hello()
```

查看当前 Shell 已经定义的所有函数，可以使用`declare`命令。

```sh
# 输出函数名和函数定义
declare -f

# 仅输出函数名
declare -F
```

#### 参数变量

函数体内可以使用参数变量，获取函数参数。函数的参数变量，与脚本参数变量是一致的。

- `$1`~`$9`：函数的第一个到第9个的参数。
- `$0`：函数所在的脚本名。
- `$#`：函数的参数总数。
- `$@`：函数的全部参数，参数之间使用空格分隔。
- `$*`：函数的全部参数，参数之间使用变量`$IFS`值的第一个字符分隔，默认为空格，但是可以自定义。

如果函数的参数多于9个，那么第10个参数可以用`${10}`的形式引用，以此类推。

`log_msg.sh`

```sh
function log_msg {
  echo "[`date '+ %F %T'` ]: $@"
}

myangvgo@pc-de-min MINGW64 /d/Dev/code/shell
$ log_msg "Application start at http://localhost:8080"
[ 2023-10-12 22:28:56 ]: Application start at http://localhost:8080
```

#### 全局变量和局部变量

Bash 函数体内直接声明的变量，属于全局变量，整个脚本都可以读取。可以使用 `local` 声明局部变量

```sh
fn () {
	local foo
	foo=1
	echo "fn: foo = $foo"
}
fn
echo "global: foo = $foo"
```

