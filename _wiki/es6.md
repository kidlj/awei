---
title: ES6
---


### 安装 Node

在 Fedora Server 23 上[1]：

	# curl --silent --location https://rpm.nodesource.com/setup_4.x | bash -
	# dnf install nodejs

### Node 对 ES6 的支持

详见这里[2]。


### 配置 babel

先安装转码规则：

	$ npm install --save-dev babel-preset-es2015

再写好 `.babelrc`：

	{
	  "presets": [
		"es2015",
	  ],
	  "plugins": []
	} 

最后安装 babel-cli：

	# npm install --global babel-cli

[1]: https://nodejs.org/en/download/package-manager/#enterprise-linux-and-fedora
[2]: https://nodejs.org/en/docs/es6/


### let

使用 `let` 声明变量有如下好处：

- 有了块级作用域；
- 不存在声明提升，在变量声明前使用变量会报错；

以后应全部使用 `let` 声明变量。


### const 命令

可以用 `const` 来声明并初始化常量，其值不可更改。

### 全局对象的属性

使用 `var` 命令和 `function` 命令声明的全局变量依旧是全局对象的属性；使用 `let`, `const` 和 `class` 命令声明的全局变量，不属于全局对象的属性。









































