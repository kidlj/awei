---
title: Regex
---


### 综述

正则表达式[wikipedia]是用来匹配某些字符串的模式表达式。

目前主要有三种正则表达式(`man grep`)：

*	Basic Regular Expressions (BRE)
*	Extended Regular Expressions (ERE)
*	Perl-Compatible Regular Expressions (PCRE)

### 元字符

有两种常见的元字符，Shell 元字符和正则表达式元字符。Shell 元字符由 Shell
解释，正则表达式元字符由各执行模式匹配操作的程序解释。两种元字符的意义不
一样。


### 基本元字符(BRE)

所有UNIX模式匹配工具都适用。

+ `^`      	匹配行首的空字符;
+ `$` 		匹配行尾的空字符；
+ `.` 		匹配任意单个字符；
+ `*` 		匹配该符号前面的最小单位的正则表达式 0 或者尽可能多次；
+ `[ ]` 	匹配集合中的任意单个字符；
+ `[^ ]` 	匹配不在该集合中的任意单个字符；
+ `[x-y]` 	匹配该字符范围内的任意单个字符；
+ `\` 		用于转义以下字符： `^  $  .  *  [`。 


### 扩展元字符(ERE)

#### 交替

+ `|`		匹配两边的任意一项

	- 该操作符有着最低优先级；
	- 如果一个表达式包含另外一个表达式，那么长的那个被匹配。

#### 子表达式

+ `()`		用于组合成子表达式；其匹配内容将被捕获。

#### 引用

+ `\1`		引用之前用 `()` 捕获的匹配内容。


#### 量词

+ `?` 		The preceding item is optional and matched at most once;
+ `+` 		The preceding item wil be matched one or more times;
+ `{n}`		The preceding item is matched exactly `n` times;
+ `{n,}`	The preceding item is matched `n` or more times;
+ `{,m}`	The preceding item is matched at most `m` times;
+ `{n, m}`	The preceding item is matched at least `n` times, but not  
		    more than `m` times.

未经修饰的量词都是「贪婪」量词。修饰量词的特性见 PCRE(`man 3 pcresyntax`)

#### 字符组

	[:alnum:]  
	[:alpha:]
	...
	[:xdigit:]

### Bracket Expressions

大多数元字符，在 `[]` 里使用时都将失去它的特殊意义，包括 `\` 在内。(`man 7 regex`)

不这里有个特例，当正则表达式用双引号括起来，而方括号内又需要填入双引号时，只能使用转义，比如：

	$ echo "ab\"cd" | grep "[\"]"


In a bracket expression,

+ To include a literal `]`, place it first in the list.
+ To include a literal `^`, place it anywhere but first.
+ To include a literal `-`, place it last.



### GNU-Specific 元字符

涉及到正则表达式的 GNU 工具普遍提供了 POSIX BRE 和 ERE 以外的一些元字符，多与单词匹配相关，列在下面；这些元字符都是借鉴于 Perl，更完整的支持见 PCRE。

#### 单词相关

+ `\<`		Matches the null string at the beginning of a word;
+ `\>` 		Matches the null string at the end of a word;  
+ `\b`		Matches the null string at the edge of a word;
+ `\B` 		Matches the null string provided it's not at the edge  
		    of a word;
+ `\w`		Is synonym for `[_[:alnum:]]`;
+ `\W`		Is synonym for `[^_[:alnum:]]`;
+ `\s`		Matches any whitespace character, shorthand for `[[:space:]]`
+ `\S` 		Matches any character that is not whitespace

Word-constituent characters are letters, digits and `_`s.

值的一提的是，因为 `\b` 在 gawk 里先作它用，因此 gawk 里用 `\y` 代替 `\b`。


### Perl 元字符(PCRE)

参见 `man 3 pcresyntax`

	# yum install pcre-devel


### 群魔乱舞

#### GNU Grep

GNU Grep 支持三种正则表达式，可以用选项指定使用哪种正则实现：

	$ grep [-G]		# 默认为BRE
	$ grep -E		# 启用ERE
	$ grep -P		# 启用PCRE


GNU Grep 中的 BRE 和 GRE 没有功能性上的差别，不过一些符号的使用需要注意(`man grep`)：

如果使用的是 BRE，那么 `?, +, {, |, (, )` 将失去其特殊意义，这些功能应该使用 `\?, \+, \{, \|, \(, \)`来替代。


#### GNU Find

GNU `find` 可以用 `-regextype type` 来指定正则表达式引擎，目前理解这些类型的正则表达式(`man find`)：

	emacs(default), posix-awk, posix-basic, posix-egrep, posix-extended

（WTF? 怎么又冒出那么多不同实现？尼玛都是 GNU 一家的就不能弄统一了？！）

#### GNU Sed

GNU Sed 使用和 GNU Grep 完全一致的 BRE 元字符。可以通过指定 `-r` 选项来使用扩展元字符 ERE。

#### GNU Awk

GNU Awk 默认使用 ERE 元字符。

#### Python

Python 的内置模块 `re` 使用 PCRE。

[wikipedia]: http://en.wikipedia.org/wiki/Regular_expression