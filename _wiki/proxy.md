---
title: Proxy
---

### Wget

    $ export http_proxy=http://192.168.116.1:1080
    $ export https_proxy=$http_proxy

### Curl

    $ curl --socks5 192.168.116.1:1080

### Yum

    # echo "proxy=http://192.168.116.1:1080" >> /etc/yum.conf

### Brew

    $ . ~/.export_proxy.sh

### Npm

    $ npm config set proxy http://192.168.116.1:1080
    $ npm config set https-proxy http://192.168.116.1:1080

### Yarn

    $ yarn config set proxy http://192.168.116.1:1080
    $ yarn config set https-proxy http://192.168.116.1:1080

### Git

    $ git config --global http.proxy http://192.168.116.1:1080
    $ git config --global https.proxy http://192.168.116.1:1080

### GitHub

    $ sudo yum install socat

    $ cat .ssh/config
    Host github.com
        HostName github.com
        User git
        # HTTP proxy(better speed!)
        ProxyCommand socat - PROXY:127.0.0.1:%h:%p,proxyport=8118
        # socks proxy
        #ProxyCommand nc -v -x 127.0.0.1:1080 %h %p

### Common

    $ export HTTP_PROXY=http://192.168.116.1:1080
    $ export HTTPS_PROXY=http://192.168.116.1:1080

### Apt

    $ cat apt_proxy.conf
    Acquire::http::proxy "http://127.0.0.1:1080";
    Acquire::ftp::proxy "http://127.0.0.1:1080";
    Acquire::https::proxy "https://127.0.0.1:1080";

    $ sudo apt-get -c ~/apt_proxy.conf update

### vscode

    "http.proxy": "http://127.0.0.1:1080",
    "http.proxyStrictSSL": false,

### Docker

    # cat /etc/systemd/system/docker.service.d/http-proxy.conf
    [Service]
    Environment="HTTP_PROXY=http://127.0.0.1:8118/" "HTTPS_PROXY=http://127.0.0.1:8118/" "NO_PROXY=localhost,127.0.0.1,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16"

### Privoxy

    # echo "forward-socks5 / 127.0.0.1:1080 ." >> /etc/privoxy/config

### Helm

    $ export HTTP_PROXY=http://192.168.116.1:1080
    $ export HTTPS_PROXY=http://192.168.116.1:1080
    $ export NO_PROXY="localhost,127.0.0.1,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16"

Note: 如果不设置 NO_PROXY 环境变量，会导致 kubectl 走代理，连接 k8s api-server 出错。
