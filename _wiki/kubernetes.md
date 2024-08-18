---
title: Kubernetes
---

### kubeadm

#### Init

    $ vagrant up

    # export IP_ADDR=10.20.9.10
    # export HOST_NAME=$(hostname -s)
    # kubeadm init --apiserver-advertise-address=$IP_ADDR --apiserver-cert-extra-sans=$IP_ADDR  --node-name $HOST_NAME --pod-network-cidr=192.168.0.0/16
    
Right now coredns (or kube-dns) is stuck in the Pending state. This is expected and part of the design.[1]

    # kubectl create -f ./calico-3.1.6.yaml

#### Join

    $ kubeadm token create --print-join-command

### Tear down[2]

    $ kubectl drain <node name> --delete-local-data --force --ignore-daemonsets
    $ kubectl delete node <node name>

    // on the node being removed
    # kubeadm reset

    // The reset process does not reset or clean up iptables rules or IPVS tables.
    # iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X
    # ipvsadm -C

### Debug failed pod

    $ kubectl describe pod <pod-name>
    $ kubectl logs <pod-name> [-p]
    $ kubectl logs calico-node-whatever -n=kube-system -c calico-node
    $ sudo journalctl -u kubelet

### ClusterIp service

ClusterIp service is not pingable[3].

### Label

    $ kubectl label node k8snode28 type=debugworker
    $ kubectl label node k8snode28 type- // remove label

### Taint

    $ kubectl taint node k8snode28 type=vipworker:NoSchedule
    $ kubectl taint node k8snode28 type:NoSchedule- // remove taint

[1]: https://kubernetes.io/docs/setup/independent/troubleshooting-kubeadm/#coredns-or-kube-dns-is-stuck-in-the-pending-state
[2]: https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/#tear-down
[3]: http://dockone.io/question/1433