---
title: Ubuntu
---


### 包管理

Debian/Ubuntu 使用 APT 作为包管理工具，而 APT 又使用更为底层的`dpkg`工具来完成工作。

更新仓库信息：

	$ sudo apt-get update

更新系统所有的包（不会额外安装或移除任何包）：

	$ sudo apt-get upgrade

安装一个包及其依赖：

	$ sudo apt-get install firefox

只更新一个包（比如受 Shellshocks 影响的 bash）：

	$ sudo apt-get install bash

移除一个包：

	$ sudo apt-get remove firefox

移除一个包及其配置文件：

	$ sudo apt-get --purge remove wpa_supplicant

更新系统所有的包（与`upgrade`不同的是，如有需要会额外安装或移除某些包）：

	$ suduo apt-get dist-upgrade

搜索描述中包含`word`的包：

	$ sudo apt-cache search word

打印某个包的详细信息：

	$ sudo apt-cache show bash

查询某个包的所有版本信息：

  $ sudo apt-cache policy postgresql-client

打印某个包的依赖：

	$ sudo apt-cache depends firefox

打印某个包的可获得版本及其依赖：

	$ sudo apt-cache showpkg firefox

列出系统中安装的所有包：

	$ dpkg --list

打印某个包的详细信息：

	$ dpkg --status bash

列出某个包安装进来的所有文件：

	$ dpkg --listfiles bash

查询某个文件是由哪个包引入：

	$ dpkg --search /bin/bash
	$ apt-file search /bin/bash

