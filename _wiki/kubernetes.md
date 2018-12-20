---
title: Kubernetes
---

kubespray
=========

### setup

安装依赖：

    # sudo pip install -r requirements.txt

初始化并启动虚拟机：

    # vagrant up

查看 ssh key：

    # vagrant ssh-config

生成 inventory：

    # cp -rfp inventory/sample/* inventory/mycluster

    # declare -a IPS=(172.17.8.101 172.17.8.102 172.17.8.103)
    # CONFIG_FILE=inventory/mycluster/hosts.ini python3 contrib/inventory_builder/inventory.py ${IPS[@]}

运行 playbook：

    # ansible-playbook -i inventory/mycluster/hosts.ini cluster.yml -b --private-key=/home/mellon/.vagrant.d/insecure_private_key -u vagrant

登录并查看：

    # vagrant ssh k8s-01
    # kubectl get nodes

关闭 swap：

    # vagrant ssh k8s-01
    # sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab


### Debug failed pod

  $ kubectl describe pod <pod-name>
  $ kubectl logs <pod-name> [-p]
  $ sudo journalctl -u kubelet


