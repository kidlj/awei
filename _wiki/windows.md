---
title: Windows
---

### Windows Build Tools

首先使用管理员身份运行 Powershell，安装 windows-build-tools：

    # npm install -g windows-build-tools

这会自动安装 Windows 构建工具和 Python 2.7。

然后设定 npm 使用的 Python 路径（node-gyp 只支持 Python 2.7）：

    $ npm config set python /c/Users/mellon/.windows-build-tools/python27/python.exe

这样就可以在 Windows 上编译安装 Node 的 native modules 了。


### 微调雅黑字体

将雅黑粗体在注册表里隐藏，会得到更好看的渲染效果：

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Fonts]
"Microsoft YaHei Bold & Microsoft YaHei UI Bold (TrueType)"="N/A"
