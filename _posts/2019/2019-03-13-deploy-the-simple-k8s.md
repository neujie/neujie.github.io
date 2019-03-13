---
layout: post
title: 一步一步，步步坚实，最简单的部署K8S玩具方法
categories:
- 学习
tags:
- K8S
---

# 准备K8S、ETCD二进制

`etcd etcdctl k8s.sh kube-apiserver kubeconfig kube-controller-manager kubectl kubelet kube-proxy kube-scheduler`

# kubeconfig文件

```bash
apiVersion: v1
kind: Config
clusters:
  - cluster:
      certificate-authority:
      server: http://10.177.243.27:8080
    name: local-up-cluster
users:
  - user:
    name: local-up-cluster
contexts:
  - context:
      cluster: local-up-cluster
      user: local-up-cluster
    name: service-to-apiserver
current-context: service-to-apiserver
```

# K8S master及master节点部署

```bash
#!/bin/bash
#
unset http_proxy; unset https_proxy
ETCD="http://｛VMIP｝:4001"
MASTERIP="｛VMIP｝"
MASTERPORT="8080"
MASTERAPI="${MASTERIP}:${MASTERPORT}"
HOSTIP="{VMIP}"
DNSIP="10.10.10.10"
BINPATH=`pwd`
LOGPATH=$BINPATH/log/
mkdir -p $LOGPATH
kill -9 `ps -elf | grep kube | grep -v grep | awk '{print $4}'`
kill -9 `ps -elf | grep etcd | grep -v grep | awk '{print $4}'`
# Master
${BINPATH}/etcd --listen-client-urls=http://0.0.0.0:4001 --advertise-client-urls=${ETCD} > ${LOGPATH}/etcd.log 2>&1 &
sleep 8
${BINPATH}/kube-apiserver --insecure-bind-address=${MASTERIP} --etcd-servers=${ETCD} --service-cluster-ip-range=10.10.10.0/24 --insecure-port=${MASTERPORT} --secure-port=0 --service-node-port-range=2000-32767 --allow-privileged=true --v=3 > ${LOGPATH}/kube-apiserver.log 2>&1 &
${BINPATH}/kube-scheduler --address=${MASTERIP} --master=${MASTERAPI} --port=10251 --scheduler-name="default-scheduler" --stderrthreshold=2 --v=3 > ${LOGPATH}/kube-scheduler.log 2>&1 &
${BINPATH}/kube-controller-manager --master=${MASTERAPI} --v=3 > ${LOGPATH}/kube-controller-manager.log 2>&1 &
# Node
${BINPATH}/kubelet --cgroup-root=/ --cluster-dns=${DNSIP} --cluster-domain=cluster.local --hostname-override=${HOSTIP} --allow-privileged=true --v=3 --kubeconfig=${BINPATH}/kubeconfig --fail-swap-on=false > ${LOGPATH}/kubelet.log 2>&1 &
${BINPATH}/kube-proxy --master=${MASTERAPI} --hostname-override=${HOSTIP} --v=3 > ${LOGPATH}/kube-proxy.log 2>&1 &
```

# 已经部署成功了

```bash
root@HGH1000049692:~/sk8s# ./kubectl -s {masterip}:8080 get nodes  -ndefault -o wide
NAME                    STATUS    ROLES     AGE       VERSION   EXTERNAL-IP   OS-IMAGE            KERNEL-VERSION                             CONTAINER-RUNTIME
ubuntu-vm-ip           Ready     <none>    3d        v1.9.2    <none>        Ubuntu 16.04 LTS    4.4.0-104-generic                          docker://18.6.1
```
