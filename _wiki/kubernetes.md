---
title: Kubernetes
---

kubeadm
=======

    $ vagrant up

    # export IP_ADDR=10.20.9.10
    # export HOST_NAME=$(hostname -s)
    # kubeadm init --apiserver-advertise-address=$IP_ADDR --apiserver-cert-extra-sans=$IP_ADDR  --node-name $HOST_NAME --pod-network-cidr=192.168.0.0/16
    
    // Right now coredns (or kube-dns) is stuck in the Pending state. This is expected and part of the design.[1]

    # kubectl create -f ./calico-3.1.6.yaml

Tear down[2]
============

    $ kubectl drain <node name> --delete-local-data --force --ignore-daemonsets
    $ kubectl delete node <node name>

    // on the node being removed
    # kubeadm reset

    // The reset process does not reset or clean up iptables rules or IPVS tables.
    # iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X
    # ipvsadm -C

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


Debug failed pod
=================

    $ kubectl describe pod <pod-name>
    $ kubectl logs <pod-name> [-p]
    $ sudo journalctl -u kubelet


Draft
=====

    $ brew install azure/draft/draft
    $ draft init
    $ draft config set disable-push-warning 1

Helm
=====

    $ brew install kubernetes-helm

In rbac-config.yaml:

    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: tiller
      namespace: kube-system
    ---
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
      name: tiller
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: cluster-admin
    subjects:
      - kind: ServiceAccount
        name: tiller
        namespace: kube-system

Install tiller:

    $ kubectl create -f rbac-config.yaml
    serviceaccount "tiller" created
    clusterrolebinding "tiller" created
    $ helm init --service-account tiller --history-max 200

### Fix failed releases

    $ helm delete --purge $RELEASE_NAME

Service
=======

### ClusterIp service

ClusterIp service is not pingable[3].

Misc
====

### Label

    $ kubectl label node k8snode28 type=debugworker
    $ kubectl label node k8snode28 type- // remove label

### Taint

    $ kubectl taint node k8snode28 type=vipworker:NoSchedule
    $ kubectl taint node k8snode28 type:NoSchedule- // remove taint

[1]: https://kubernetes.io/docs/setup/independent/troubleshooting-kubeadm/#coredns-or-kube-dns-is-stuck-in-the-pending-state
[2]: https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/#tear-down
[3]: http://dockone.io/question/1433