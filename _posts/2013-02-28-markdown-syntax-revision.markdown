---
layout: post
title: Markdown书写语言语法说明笔记
---


Markdown书写语言语法说明笔记
===========================


概述
----

### 宗旨 ###

Markdown的目标是成为一种好用的适用于网络的**书写**语言。

### 与HTML兼容 ###

Markdown不是要取代HTML，甚至连接近都算不上，它的语法种类很少，
只对应HTML的一部分。虽然Markdown经过处理后是要变成HTML文档，
但Markdown的构想不是使HTML容易书写，HTML已经很容易书写了。
Markdown的理念是能够让文档更容易读、写和修改。HTML是一种发布
格式，而Markdown是一种书写格式。

因此，Markdown的格式语法只涵盖纯文本处理的范围。不在Markdown
涵盖范围内的标签，都可以直接在文档里用HTML书写，不需要加注
这是HTML还是Markdown, 直接用这些标签就可以了。

在Markdown文本内使用HTML标签需要**注意**：  

+   应用HTML某些区块标签(block-level elements)，比如`<div>`, `<table>`
    等标签时，必须在前后加上空行与其他内容隔开，还要求他们的
	开始标签与结尾标签不能用制表符或空格缩进。

+   HTML区块标签内的Markdown语法将不会被处理。
    举例如下(`*Foo*`不起作用):

    这是一个普通段落。

	     <table>
	    	 <tr>
				 <td>*Foo*</td>
			 </tr>
		 <table>

    这是另一个普通段落。  

+   HTML的区段(span-level elements)可以应用于Markdown文本的
    任意部分。而且Markdown语法在HTML区段标签内是有效的。

### 特殊字符自动转换 ###

在HTML中，有两个字符需要特殊处理，`<` 和 `&`, 如果要显示这两个字符
的原型，必须要使用实体形式(entities), 分别是`&lt;` 和 `&amp;`
Markdown会自动处理`<`和`&`的显示。如果`<`或`&`与其后的字符构成具有特殊
意义的HTML标识(如`&amp; &lt; <href>`等)则不予变动;否则就将`<`和`&`当作HTML
实体而予以自动转换(变成`&amp;`等)。

比如：

	4 < 5

会自动变成

	4 &lt; 5

**注意**:(?)Markdown code范围内，不论是区块还是区段(行内), `<`和`&`
两个符号一定都会被转换成HTML实体，这项特性使得很容易用Markdown
写HTML code(如果在HTML文档中引用HTML code，需要将所有的`<`和`&`进行
实体转换(即code内所见即所得)。

区块元素(Block-level Elements)
------------------------------

###段落和换行(Paragraphs and Line Breaks)###

一个Markdown段落是由一个或多个连续的文本行组成，它的前后要有一个
以上的空行。普通段落不要缩进。

一个段落里的文本可以强制断为多行(hard-wrapped,相当于HTML里在上一行结尾
处插上`<br />`), 方法是在插入处先按入两个以上的空格然后回车。

### 标题(Headers) ###

Markdown支持两种标题(Headers)的写法。

	This is H1
	==========

	This is H2
	----------

或者
	
	# This is H1 #
	## This is H2 ##
	### This is H3 ###
	...
	###### This is H6 #

### 区块引用(BlockQuotes) ###

Markdown的区块引用类似于Email中的`>`引用方式。将所引用段落手动断行
(hard-wrap)并且在每一行加上`>`效果最好。

>这是一个测试。你可以大胆地手工断行排版，而不用担心引入
>`<br />`，就像这样。

>>区块引用也可以嵌套。
>>就像这样。

>OK. It works.(这一行与上一行内容必须有空行隔开)。在这里我们断行，
>这是效果示例，语法示例请查看源文件。

引用的区块内也可以使用其它的Markdown语法，包括标题、列表、代码区块等。

> #### 这是一个四级标题 ####
>
> 1. 这是第一个列表项。
> 2. 这是第二个列表项。
>
> 一些代码示例(代码区块缩进8个空格,同下列表内代码区块缩进):
>
>		$ grep '[a-z]' Regex.txt -o  
>		$ grep 'test' Regex.txt
>
>		$ grep '[a-z]*' Regex.txt -o

### 列表(Lists) ###

Markdown支持无序列表和有序列表。

无序列表使用`*`, `+` 或是`-` 作为列表标记：

* Babel
* Sigh No More
+ Tago Mago
+ Can
+ Future Days
+ Saw Delight
- Marquee Moon
- Adventure
- Television

有序列表使用一个数字接着一个数点(number-period-spaces squence)。

1. Mumford & Sons
2. Can
3. Television

列表标识符最多可以缩进3个空格。数点后要留有至少一个空格。

一个列表项可以包含多个段落，该项目下的每个段落都必须缩进4个空格。

如果在列表项里放进引用区块，`>`需要缩进4个空格：

* A list item with a blockquote:

  	> This is a blockquote  
  	> inside a list item.

如果在列表项里放进代码区块，该代码区块需要缩进两次，即8个空格：

*   A list item with a codeblock:

		$ ls -l .

项目列表很可能会不小心产生，像是下面这样的写法:

    2013. what a tough year.

要避免这样的情况，可以在句点前面加上反斜杠，如下：

	2013\. What a tough year.

2013\. What a tough year.

### 代码区块(CodeBlocks) ###

Markdown会同时用`<pre><code>`将代码区块包起来。

要在Markdown里建立代码区块只需简单地缩进4个空格就可以。

这是一个普通段落。

	This is a code block.	
	This is another code block.
	And this line is not hard-wrapped(with 2+ spaces).

这是另一个普通段落。

**注意**：Code(and Lists)标记(难道凡是区块标记,不论是Markdown里还是HTML里的?)要想在Markdown 
文档里起作用，其前后必须用空行分隔，与前面“与HTML兼容”章节**注意** 
所述内容相契合。

缩进用的4个空格只起标示作用，在转换成HTML后会被剥落，不起作用。

一个代码区块会一直持续到没有缩进的那一行(或是文件结尾)。

在代码区块里，`&`, `<`和`>`会自动转成HTML实体，方便在Markdown里插入HTML
语句。同理，代码区块里Markdown标记也不起作用，这意味着在Markdown里
插入Markdown语法很方便。

### 分隔线 ###

这些都可以构成分隔线：

	* * *  
	- - -  
	_ _ _

区段元素(Span-level Elements)
-----------------------------

### 链接(Links) ###

Markdown支持两种形式的链接，内联式和引用式。

不管是哪一种，链接文字都是用[方括号]来标记。

内联式：

	[Example](http://www.example.com "Optional Title")

效果：[Example](http://www.example.com "As an example")

参考式：

	[Example][id]

	[id]: http://www.example.com "Optional Title"

也可以再简单一些：

	[Example][]

	[Example]: http://www.example.com "optional"

效果：I get 10 times more traffic from [Google][] than from 
[Baidu][1] and [Bing](http://www.bing.com "Bing Search")

[Google]: http://www.google.com "Google Search"
[1]: http://www.baidu.com "Baidu Search"
(这些定义在转成HTML时将会被剥落。)

### 强调(Emphasis) ###

*single asterisks*  

_single underscores_

**double asterisks**   

__double underscores__

or like this: Mumford **&** Sons.

如果要在文字前后插入普通的星号或底线，可以这样：

	\*This text is surrounded by literal asterisks\*

效果：\*This text is surrounded by literal asterisks\*

### 代码(Code) ###

如果要标记一小段行内代码，可以用反引号(`)把它包起来，例如：

Use the `printf()` function.

如果要在代码区段内插入反引号，你可用多个反引号来开启和结束
代码区段，比如：

``There is a literal backtick (`) here. ``

A single backtick in a code span: `` ` ``

A backtick-delimited string in a code span: `` `foo` ``

在代码区段内，<和&都会被自动转成HTML实体，这使得在Markdown
文档中插入HTML原始码很容易。

Please do not use any `<blink>` tags.

`&#8212` is the decimal-decoded equivalent of `&mdash;`.

### 图片(Images) ###

Markdown使用一种和链接很相似的语法来标记图片，同样分为
内联式和引用式两种。

内联式：

	![Alt](/path/to/img.jpg)  
	![Alt](/path/to/img.jpg "Optional Title")

引用式：

	![Alt][id]

	[id]: /path//to/img.jpg "Optional Title"

目前为止，Markdown尚不能指定图片的宽高，如果需要可以使用
HTML的`<img>`标签。

其它(Miscellaneous)
-------------------

### 自动链接(Automatic Links) ###

<http://www.google.com>

<kdlijian@gmail.com>

### 反斜杠转义(Backslash Escapes) ###

Markdown支持在以下这些字符的前面插入反斜杠来输入普通字符：

`\ * _ - + [] {} () # . !`和\`

-EOF-    2013/2/28

### 疑问 ###

`<`和`<`在markdown语法里有什么特殊意义？为什么有时候会改变Markdown
文本的颜色？

### test gist ###

{% gist 5555251 %}
