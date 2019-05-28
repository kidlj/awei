---
title: Docker
---

### Theory

每一条指令(Instruction) 在 image 上增加一层。

Docker build 的运行过程是，由当前的 image 创建出 container，进入该 container 运行命令，然后 commit 变更生成一个新的 image(也就是在之前的 image 上增加一层)，随后删除刚才用过的临时 container。

接着往下按照这种 workflow 运行下一条指令，直到最后生成一个最新的 image(多层中间 image 累积的结果）。

### Daemon

    # ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock

    # mkdir -p /etc/systemd/system/docker.service.d/
    # cat /etc/systemd/system/docker.service.d/http-proxy.conf 
    [Service]
    Environment="HTTP_PROXY=http://master:8118"
    Environment="HTTPS_PROXY=http://master:8118"

    # systemctl daemon-reload
    # systemctl restart docker

### Client

    $ export DOCKER_HOST="tcp://lijian-k8s-node1:2375"
    $ docker version

### Image

    $ docker image ls | head
    $ docker image tag 3cdd5f4c931c kidlj.com/echo-demo:1.0.0
    $ docker image save kidlj.com/echo-demo:1.0.0 | gzip -c > echo-demo-1.0.0.tar.gz
    $ docker image load -i ./echo-demo-1.0.0.tar.gz