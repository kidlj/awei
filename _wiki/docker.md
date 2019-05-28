---
title: Docker
---

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