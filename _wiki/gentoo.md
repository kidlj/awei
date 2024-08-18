---
title: Gentoo
---


### Custom Ebuilds

可以很简单地建立一个本地 overlay，用于存放各种分散的 ebuilds。

举个例子，想要安装 `fcitx-sogoupinyin` 的话，可以添加 `gentoo-zh` 这个 overlay，这是标准做法。可是这个 overlay 太大了，如果仅仅想要安装这一个包该怎么办呢，这时就可以用建立本地 overlay 的方法了。

1. 建立本地 overlay

		$ sudo mkdir /var/lib/layman/custom
		$ sudo mkdir /var/lib/layman/custom/profiles
		$ sudo echo "custom" > /var/lib/layman/custom/profiles/repo_name

	然后在 `/var/lib/layman/make.conf` 文件里适当的位置加上一行，
		
		/var/lib/layman/custom

	这样这个叫做 `custom` 的 overlay 就建好了。

2. 放置 ebuild

	下面的工作就是把第三方或者自己维护的 ebulds 放置在这个 overlay 里，以 `fcitx-sogoupinyin` 为例：

		$ cd /var/lib/layman/custom
		$ sudo mkdir -p app-i18n/fcitx-sogoupinyin/
		$ cd app-i18n/fcitx-sogoupinyin/
		$ cp ~/fcitx-sogoupinyin-0.0.6.ebuld .
		
	然后重要的一步，为该 ebuild 生成 Manifest 文件：
		
		$ ebuld fcitx-sogoupinyin-0.0.6.eduild digest

	如果成功，该 ebuild 就可以通过 Portage 使用了。

3. 安装 package

		$ sudo emerge -av fcitx-sogoupinyin


