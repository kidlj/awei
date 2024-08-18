---
title: Shell
---

### 变量

#### 引用

引用一个变量必须在前面加上`$`：比如`echo $PATH`；还有一种更正式的引用方式为`${PATH}`。

#### 初始化

Shell 变量不需要初始化。  
引用一个未定义（赋值）的变量不会抛出错误，没有被赋值的变量为空字符串。

#### 变量类型

Shell 变量没有“类型”的概念，如果一定要说类型，那么任何变量都是字符串。但有些函数在处理字符串时会将其中包含的数字字符当成数值来处理，这为 Shell 的数学运算提供了便利。

#### 作用域

与大多数编程语言不同，Shell 函数内的变量在函数外部可以访问，其值也得到保留，就相当于其它语言的全局变量。

如果非要使用函数内的“局部变量”，那么在变量前加上 `local`。

#### 命名规则

通常而言，包含常数、字符串以及文件名等内容的变量使用大写，而包含数字、用户输入或其他“数据”的变量则使用小写。

#### 赋值

对变量赋值的方式主要有三种：

- 显式定义：`VAR=value`

	等于号两边不允许有空格。像`VAR = value`，`VAR =value`以及`VAR= value`都不是赋值语句，而有着各自的含义。`VAR = value`被当成带有两个参数的命令`VAR`，参数是`=`和`value`；`VAR =value`同理。`VAR= value`是指在执行`value`命令的时候将`VAR`变量赋值为空。

- 读取：`read VAR`

	一次读入一个变量：
		
		echo -n "Enter you name: "
		read name
		echo "Hello $name."
		
	一次读入多个变量：

		echo -n "Please enter your first name and last name: "
		read firstname lastname
		echo "Hello, $firstname. How is the $lastname family?"

	用一个变量从文件中读入一行：

		read message < /var/lib/portage/world
		echo $message

	用一个变量输出整个文件，每次打印一行：

		while read message
		do
			echo $message
			sleep 0.5
		done < /var/lib/portage/world

	（注：如果读取到文件末尾，`read`会返回非零值，于是 while 循环就会停止。）

- 命令替换：`VAR=$(date)`

	这是第一种赋值方式 `VAR=value` 的变种。

		TODAY=$(date +%A)
		echo "Today is $TODAY."

	还有一种更通用的命令替换格式，原始的 Bourne Shell 只支持这种：

		TODAY=`date +%A`

	（注：命令替换会开启一个子 shell 来执行括号中的命令。）

#### 删除变量

可以用`unset`命令删除一个变量：

	$ unset var

或者直接将变量赋值为空：

	$ var=

不过这两者本质上不相同的。后者虽然将变量赋值为空，但其仍然存在；而`unset`命令会真正的删除一个变量。


#### 参数扩展

当两边加上`" "`时：

- `"$*"` 扩展为 `"$1c$2c..."`，这里 `c` 指 `IFS` 的第一个字符。  
- `"$@"` 扩展为 `"$1" "$2" ...`。

### 高级变量操作

#### 字符串裁剪

* 按照长度裁剪变量字符串

		$ variable="foobar"

		$ echo ${variable:3}
		bar
		$ echo ${variable:3:2}
		ba

		$ echo {variable: -4}
		obar

* 使用模式裁剪字符串

		$ phone="185-1111-2222"

		$ echo ${phone#*-}
		1111-2222
		$ echo ${phone##*-}
		2222

		$ echo {phone%-*}
		185-1111
		$ echo {phone%%-*}
		185

#### 变量字符串的长度

可以获取一个变量的字符串长度：

	$ var="Test"
	$ echo ${#var}
	4

#### 大小写转换

可以将变量内容转为全部大写或者小写：

	$ var="Test"
	$ echo ${var^^}
	TEST
	$ echo ${var,,}
	test

#### 提供缺省值

在引用变量时，为避免变量值为空，可以为其指定缺省值：

	$ ${EDITOR:-/usr/bin/vim} "$myfile"
	$ ${EDITOR:-`which vim`} "$myfile"

可能每次都这样做会比较麻烦，可以在指定变量缺省值的同时将其赋值：

	$ ${EDITOR:=/usr/bin/vi} "$myfile1"
	$ ${EDITOR} "$myfile2"

#### 基于变量存在的操作

如果一个变量有值（即使被赋为空值），则可基于此做一些有趣的事情：

	$ VILOVER=yes
	$ ${VILOVER+"vi"} "$file"

此时将会启动`vi`程序编辑`$file`文件。

#### 间接变量

运行如下代码：

	$ for myvar in PATH HOSTNAME; do echo $myvar is ${!myvar}; done

结果返回：

	PATH is /sbin:/usr/sbin:/usr/bin:/usr/local/bin:/usr/bin:/bin
	HOSTNAME is collie

也可以对间接变量赋值：

	$ abc=xyz
	$ eval ${!abc}=test

到底是什么意思就自己想吧 :-) 


#### 字符串查找与替换

Sed 工具提供了强大的文本查找替换功能，除了调用 sed 以外，Shell 本身也具有基本的查找和替换功能。

准备数据至变量：

	$ user=`grep mellon /etc/passwd`
	
可以只替换第一个和全局替换：
	
	$ echo ${user/mellon/collie}
	$ echo ${user//mellon/collie}

也可以指定模式匹配是在开头还是结尾。正则表达式用`^mellon`表示处在开头处    的匹配，而 Shell 不使用正则表达式，分别用`#`和`%`来表示头部和尾部的模式    匹配：

	$ echo ${user/^mellon/collie}
	$ echo ${user/%bash/csh}

这里也可以使用 Shell 通配符来匹配查询：

	$ echo ${user/me??on/collie}

如果省略掉要替换进来的内容，则将删除匹配的文本：

	$ echo ${user/mellon}
	$ echo ${user//mellon}
	$ echo ${user/#mellon}


### 数学运算

虽然 Shell 变量始终被存储为字符串，但是可以用一些方法使它能像数字一样运算。

- `let`

		let result=x+y
		let no1++
		let no2--
		let no+=6
		let no*=4

- `$(( ))`

		result=$((x + y))
		result=$((x * y))
		result=$((x / y))
		result=$((x % y))
		result=$((x ** y))
		result=$((x++))


注：以上这些方法只能用于整数运算，而不支持浮点数。



### 文件描述符与重定向

#### 重定向

分别将`stdout`和`stderr`重定向：

	$ ls -l /home /nosuchfile 1> /tmp/stdout.txt 2> tmp/stderr.txt

同时将`stdout`和`stderr`重定向到一个文件：

	$ ls -l /home /nosuchfile > /tmp/output.txt 2>&1
	$ ls -l /home /nosuchfile >& /tmp/output.txt
	$ make install 2>&1 | tee make_install.log

输出`stdout`和`stderr`的同时，将`stdout`重定向到一个文件：

	$ ls -l /home /nosuchfile | tee output.txt


#### 管道

管道将前一个进程的`stdout`连接到另一个进程的`stdin`，而`stderr`仍然输出到终端或者可以重定向到文件：

	$ find / -print | grep hosts

该命令会混合输出`grep hosts`的结果以及`find`命令的错误。可以将`find`命令的`stderr`提前重定向：

	$ find / -print 2> /dev/null | grep hosts

#### 数据流输出

输出重定向时，清楚地选择使用 `>` 还是 `>>`：

- `find` 命令的输出只有一个数据流

		$ find . -type f -name '*.sh' -exec cat {} \; > all_sh_files.txt

-  循环的输出只有一个数据流

		$ for i in {1..5}; do echo $i; done > /tmp/test.txt



### 正则表达式与引用

正则表达式与 shell 没有直接关系，只有像 grep, sed, awk 这样的外部命令才使用正则表达式。但因为 shell 扩展和正则表达式使用的语法非常相似，所以从 shell 传递给外部命令的正则表达式可能会在中途被 shell 分析。这一问题可以使用各种引用技术来避免。

有三种主要形式的引用—— 单引号、双引号以及反斜线。

#### 单引号

单引号可以将它中间的所有任意字符还原为字面意义，包括 `\` 在内。因此在两个单引号中间包括一个单引号是不可能的。

	$ echo '\''		# bad code

#### 双引号

双引号引用，不会屏蔽反引号、`\` 和 `$` 三个元字符。其它的包括单引号在内还原为字面意义。

	$ echo "\\"
	\
	$ echo "\" 		# bad code

比如 `sed` 的表达式经常用单引号扩起来，但当我们想在 `sed` 表达式里使用一些变量时，双引号就能派上用场了。

	$ text=hello
	$ echo hello world | sed "s/$text/HELLO/"
	HELLO world

#### 反斜线

当需要在常规字符串中包含特殊字符，但它又会被 shell 解释的时候，我们可以在字符前加上反斜线。

	$ echo "Wilde said, \"Experience is one thing you can't get for\
	 nothing.\""

在双引号中，反斜线起转义作用；在单引号中不起转义作用。
	

### 数组

#### 数组的赋值

- 用索引

		numberarray[0]=zero
		numberarray[1]=one
		numberarray[3]=three

- 用括号

		students=( Dave Jennifer Michael Lucy Richard )
		nonprint=( [0]=NUL [1]=SOH [2]=STX [4]=EOT )
		stat=( `cat /proc/$$/stat` )
		mp3s=( *.mp3 )

- 用 read

		read -a arrayname
		IFS=: read -a userdetails < /etc/passwd
		readarray -n 4 -s 2 food

#### 数组的访问

用索引访问：

	echo ${arrayname[0]}

打印数组中所有的值：

	echo ${arrayname[@]}

打印数组长度：

	$ echo ${#arrayname[@]}

#### 数组迭代

    $ images=(7a2bc5dc2085 99bd113dbe1e 3cdd5f4c931c)
    $ for key in ${!images[@]; do echo ${images[key]}; done

#### 关联数组

定义：

	declare -A a_array
	a_array=( [index1]=value1 [index2]=value2 )

访问：

	echo ${a_array[index1]}
	echo ${a_array[@]}

列出索引：

	echo ${!a_array[@]}


### 比较与测试

`test`是一个 shell 内置命令，它的另一个名字是`[`。同时操作系统还有`/usr/bin/test`存在，不过 shell 优先找到的是其内置命令。

字符串比较测试是，应优先使用`[[`而不是`[`，因为前者更强大和规范。而考虑到 Bourne Shell 兼容性时，应使用后者。

#### 算术比较

一定要在`[`或`]`与操作数之间有一个空格。

* `[ $var -eq 0 ]`
* `[ $var -ne 0 ]`
* `[ $var -gt 0 ]`
* `[ $var -ge 0 ]`
* `[ $var -lt 0 ]`
* `[ $var -le 0 ]`

#### 文件系统相关测试

* `[ -f $file_var ]`
* `[ -d $file_var ]`
* `[ -e $file_var ]`
* `[ -r $file_var ]`
* `[ -s $file_var ]`

#### 字符串比较测试

使用`[[`：

* `[[ $str1 == $str2 ]]`
* `[[ $str1 != $str2 ]]`
* `[[ $str1 > $str2 ]]`
* `[[ -z $var ]] # when empty, return True`
* `[[ -n $var ]] # when not empty, return True`

使用`[`：

* `[ $str1 = $str2 ]`
* `[ $str1 != $str2 ]`

#### 正则表达式测试

Bash 3 新增的一个特性是`=~`操作符，但应使用`[[`：

	if [[ $pkgname =~ .*\.deb ]]
	then
		echo "File $pkgname is a .deb package
	fi


#### 检验返回值

Shell 里的条件执行只能通过检查条件表达式的返回值来决定真假，如果条件表达式（也可以是某条 shell 命令）返回 0, 则为真，否则为假。

跟其它编程语言不一样，shell 不可以通过检查表达式自身是否为空来决定真假。比如 Python 里，条件表达式可以是一个变量，如果该变量为空则表达式值为假，不为空则表达式值为真，这在 shell 里是行不通的。 Shell 总要执行条件表达式，然后检查其返回值。

所以，在 Python 中，我们可以说 0 或者空就是假，非 0 非空就是真；在 shell 中不能这样说，只能说一个命令返回 0 就是真，返回非 0 就是假。

值得一提的，`true` 命令和 `:` 都会简单地返回 0, 但是因为后者内置于 shell 中所以执行效率更高。


### IFS 与迭代器

IFS 的默认值是空白字符（换行符、制表符或者空格）。

可以使用 IFS 读取变量中每一个条目：

	data="name,sex,location,age"
	oldIFS = $IFS
	IFS = ","
	for item in $data
	do
		echo Item: item
	done
	IFS=$oldIFS

使用`for var in list`迭代时，`list`可以是字符串，也可以是一个序列。

可以轻松地生成不同的序列：

* `{1..50}`
* `{a..z}`


