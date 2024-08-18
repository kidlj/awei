---
title: GnuPG
---


### 加密和签名原理

加密和解密的过程是，用接收者的公钥（公开）加密信息，对方用对应的私钥解密。

验证签名的过程是，由发送者的私钥和要发送的信息内容生成签名，接收者需使用发送者的公钥验证签名。

### 密钥管理

生成钥匙对：

	$ gpg --gen-key

列出本地的公钥：

	$ gpg --list-keys

将自己的公钥导出(`www.ideavirgin.com`为我的公钥UID）：

	$ gpg --armor --output pubkey.txt --export www.ideavirgin.com

把公钥发布到公钥服务器(`E70B1BE1`为我的公钥ID）：

	$ gpg --send-key E70B1BE1

生成吊销证书：

	$ gpg --output revoke.asc --gen-revoke E70B1BE1

导入他人的公钥，有多种形式：

	$ gpg --search-key chet@cwru.edu
	$ gpg --recv-key E70B1BE1
	$ gpg --import keyname.asc

核对第三方公钥的指纹值并用自己的私钥签收之：

	$ gpg --fingerprint chet@cwru.edu
	$ gpg --sign-key chet@cwru.edu

### 加密与解密

用某人的公钥加密文件：

	$ gpg --recipient www.ideavirgin.com --output demo.en.txt --encrypt demo.txt

用对应的私钥解密文件：

	$ gpg --output demo.de.txt --decrypt demo.en.txt

### 签名

将签名连同文件内容一起生成一个新文件：

	$ gpg --armor --clearsign demo.txt
	$ gpg --verify demo.txt.asc

将签名与文件内容分开：

	$ gpg --armor --detach-sign demo.txt
	$ gpg --verify demo.txt.asc

### 签名验证举例

GNU ftp 里为发布的文件提供了分离式验证签名和验证公钥，须如下验证签名:

	$ wget http://ftp.gnu.org/gnu/bash/bash-4.3-patches/bash43-001
	$ wget http://ftp.gnu.org/gnu/bash/bash-4.3-patches/bash43-001.sig
	$ wget http://ftp.gnu.org/gnu/gnu-keyring.gpg
	$ gpg --keyring ./gnu-keyring.gpg --verify bash43-001.sig

