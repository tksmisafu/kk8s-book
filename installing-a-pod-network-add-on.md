---
description: >-
  https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/#pod-network
---

# Installing a pod network add-on

### network add-on flannel

我選擇 flannel 作為 K8s CNI 功能實現方式。 有關 K8s 官方說明，可以查看此篇 [https://kubernetes.io/docs/concepts/cluster-administration/networking/\#flannel](https://kubernetes.io/docs/concepts/cluster-administration/networking/#flannel)   
Github flannel 連結  
[https://github.com/coreos/flannel](https://github.com/coreos/flannel)

因我對 CNI 尚未深入了解，故此篇僅偏向實作筆記，沒有談論到 CNI 相關面向資訊。  
當完成了 kubeadm init 步驟後，下一步驟就是安裝 K8s CNI 角色

```bash
You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/
```

### 安裝 flannel

```bash
# 選擇安裝 flannel
# kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml

[vagrant@kk8s-1 ~]$ kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
serviceaccount/flannel created
configmap/kube-flannel-cfg created
daemonset.extensions/kube-flannel-ds-amd64 created
daemonset.extensions/kube-flannel-ds-arm64 created
daemonset.extensions/kube-flannel-ds-arm created
daemonset.extensions/kube-flannel-ds-ppc64le created
daemonset.extensions/kube-flannel-ds-s390x created
[vagrant@kk8s-1 ~]$
```

#### 查看有關 CoreDNS 及 flannel pod 運作狀況

```bash
[vagrant@kk8s-1 ~]$ kubectl get pods --all-namespaces
NAMESPACE     NAME                             READY   STATUS             RESTARTS   AGE
kube-system   coredns-576cbf47c7-jjd9m         1/1     Running            0          18h
kube-system   coredns-576cbf47c7-vsldw         1/1     Running            0          18h
kube-system   etcd-kk8s-1                      1/1     Running            0          18h
kube-system   kube-apiserver-kk8s-1            1/1     Running            1          18h
kube-system   kube-controller-manager-kk8s-1   1/1     Running            0          18h
kube-system   kube-flannel-ds-amd64-xm4hc      0/1     CrashLoopBackOff   5          5m11s
kube-system   kube-proxy-mwrnv                 1/1     Running            0          18h
kube-system   kube-scheduler-kk8s-1            1/1     Running            1          18h

##
# 發現 pod STATUS：CrashLoopBackOff
##
```

### 出錯，解決問題

#### 查看 log

```bash
[vagrant@kk8s-1 ~]$ kubectl -n kube-system logs kube-flannel-ds-amd64-xm4hc
I1112 12:23:10.375512       1 main.go:475] Determining IP address of default interface
I1112 12:23:10.377462       1 main.go:488] Using interface with name eth0 and address 10.0.2.15
I1112 12:23:10.377540       1 main.go:505] Defaulting external address to interface address (10.0.2.15)
I1112 12:23:10.483847       1 kube.go:131] Waiting 10m0s for node controller to sync
I1112 12:23:10.483997       1 kube.go:294] Starting kube subnet manager
I1112 12:23:11.485223       1 kube.go:138] Node controller sync successful
I1112 12:23:11.485254       1 main.go:235] Created subnet manager: Kubernetes Subnet Manager - kk8s-1
I1112 12:23:11.485260       1 main.go:238] Installing signal handlers
I1112 12:23:11.485325       1 main.go:353] Found network config - Backend type: vxlan
I1112 12:23:11.485388       1 vxlan.go:120] VXLAN config: VNI=1 Port=0 GBP=false DirectRouting=false
E1112 12:23:11.485688       1 main.go:280] Error registering network: failed to acquire lease: node "kk8s-1" pod cidr not assigned
I1112 12:23:11.485737       1 main.go:333] Stopping shutdownHandler...
[vagrant@kk8s-1 ~]$

```

{% hint style="danger" %}
Error registering network: failed to acquire lease: node "kk8s-1" pod cidr not assigned
{% endhint %}

經過爬文得知，要使用此網路解決方案，則需要再 kubeadm init 時加上   
`--pod-network-cidr=10.244.0.0/16`  
我又爬了文，想嘗試現行環境下在某個設定檔案中追加此參數，但沒有查到此解法，看來我只能 reset master node。

#### 記錄移除了整個 master node 過程

```bash
以下，我完整記錄了移除整個 master node 的過程
[vagrant@kk8s-1 ~]$ kubectl -n kube-system delete deployments --all
deployment.extensions "coredns" deleted

[vagrant@kk8s-1 ~]$ kubectl delete daemonsets -n kube-system --all
daemonset.extensions "kube-flannel-ds-amd64" deleted
daemonset.extensions "kube-flannel-ds-arm" deleted
daemonset.extensions "kube-flannel-ds-arm64" deleted
daemonset.extensions "kube-flannel-ds-ppc64le" deleted
daemonset.extensions "kube-flannel-ds-s390x" deleted
daemonset.extensions "kube-proxy" deleted

[vagrant@kk8s-1 ~]$ kubectl -n kube-system delete services --all
service "kube-dns" deleted

[vagrant@kk8s-1 ~]$ kubectl -n kube-system delete pods --all
pod "etcd-kk8s-1" deleted
pod "kube-apiserver-kk8s-1" deleted
pod "kube-controller-manager-kk8s-1" deleted
pod "kube-flannel-ds-amd64-xm4hc" deleted
pod "kube-proxy-mwrnv" deleted
pod "kube-scheduler-kk8s-1" deleted
[vagrant@kk8s-1 ~]$

###
# 以上我刪除了 namespace = kube-system 的 services \ pods，都還是自動恢復服務，
# 我沒有找到原因，就直接往下進行 reset 囉～
###
[vagrant@kk8s-1 ~]$
[vagrant@kk8s-1 ~]$ sudo kubeadm reset
[reset] WARNING: changes made to this host by 'kubeadm init' or 'kubeadm join' will be reverted.
[reset] are you sure you want to proceed? [y/N]: y
[preflight] running pre-flight checks
[reset] stopping the kubelet service
[reset] unmounting mounted directories in "/var/lib/kubelet"
E1112 13:46:49.748364   12376 reset.go:145] [reset] failed to remove containers: docker is required for container runtime: exec: "docker": executable file not found in $PATH
[reset] deleting contents of stateful directories: [/var/lib/kubelet /etc/cni/net.d /var/lib/dockershim /var/run/kubernetes /var/lib/etcd]
[reset] deleting contents of config directories: [/etc/kubernetes/manifests /etc/kubernetes/pki]
[reset] deleting files: [/etc/kubernetes/admin.conf /etc/kubernetes/kubelet.conf /etc/kubernetes/bootstrap-kubelet.conf /etc/kubernetes/controller-manager.conf /etc/kubernetes/scheduler.conf]
[vagrant@kk8s-1 ~]$
[vagrant@kk8s-1 ~]$ sudo kill 595 654
[vagrant@kk8s-1 ~]$ sudo systemctl restart kubelet

```

### 重新 kubeadm init

```bash
[vagrant@kk8s-1 ~]$ sudo kubeadm init --cri-socket="/var/run/crio/crio.sock" --apiserver-advertise-address=192.168.42.191  --pod-network-cidr=10.244.0.0/16

[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join 192.168.42.191:6443 --token noduzf.t2ng9nhcx2gdo53y --discovery-token-ca-cert-hash sha256:d02f588e50c1f16fb9eec512e5ca14fd2add98e0231805a46c4e0fb2aff83555

```

#### 重新安裝

```text
[vagrant@kk8s-1 ~]$ kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
serviceaccount/flannel created
configmap/kube-flannel-cfg created
daemonset.extensions/kube-flannel-ds-amd64 created
daemonset.extensions/kube-flannel-ds-arm64 created
daemonset.extensions/kube-flannel-ds-arm created
daemonset.extensions/kube-flannel-ds-ppc64le created
daemonset.extensions/kube-flannel-ds-s390x created

[vagrant@kk8s-1 ~]$ kubectl get pods --all-namespaces
NAMESPACE     NAME                             READY   STATUS              RESTARTS   AGE
kube-system   coredns-576cbf47c7-d2c46         0/1     ContainerCreating   0          2m29s
kube-system   coredns-576cbf47c7-wk4tg         0/1     ContainerCreating   0          2m29s
kube-system   etcd-kk8s-1                      1/1     Running             1          91s
kube-system   kube-apiserver-kk8s-1            1/1     Running             2          109s
kube-system   kube-controller-manager-kk8s-1   1/1     Running             0          93s
kube-system   kube-flannel-ds-amd64-6dmw9      1/1     Running             0          43s
kube-system   kube-proxy-hrsdw                 1/1     Running             0          2m28s
kube-system   kube-scheduler-kk8s-1            1/1     Running             2          88s

###
# 什麼 coredns-576cbf47c7-d2c46  <STATUS>  ContainerCreating.....
# 發現 coredns pod 一直失敗
###
```

### 又卡關

#### 先查 log

```bash
kubectl -n kube-system describe pods coredns-576cbf47c7-d2c46
Events:
  Type     Reason                  Age                   From               Message
  ----     ------                  ----                  ----               -------
  Normal   Scheduled               6m50s                 default-scheduler  Successfully assigned kube-system/coredns-576cbf47c7-d2c46 to kk8s-1
  Warning  FailedCreatePodSandBox  6m49s                 kubelet, kk8s-1    Failed create pod sandbox: rpc error: code = Unknown desc = failed to create pod network sandbox k8s_coredns-576cbf47c7-d2c46_kube-system_9d02ffe4-e687-11e8-8b3f-525400c9c704_0(6084cd73561cec86165ac1f749de7621ce4f6641b6a5e1af1425094a87e1027f): failed to set bridge addr: "cni0" already has an IP address different from 10.244.0.1/24
  Warning  FailedCreatePodSandBox  79s (x17 over 6m54s)  kubelet, kk8s-1    (combined from similar events): Failed create pod sandbox: rpc error: code = Unknown desc = failed to create pod network sandbox k8s_coredns-576cbf47c7-d2c46_kube-system_9d02ffe4-e687-11e8-8b3f-525400c9c704_0(8b8ab802339a3ccaf6bcf3bc6cf162a6bbe5ebdcbbd7369e163e8fc6977ce763): failed to set bridge addr: "cni0" already has an IP address different from 10.244.0.1/24
  
###
# 重點就是，網卡無法綁定此網段 10.244.0.1/24 IP
###
```

經過透過 Google 搜尋前人經驗，發現在建立 flannel 時會創建`cni0、flannel.1`兩網卡。  
找到原因了~ 原來網卡衝突了！  
會衝突的原因是，我在相同環境反覆 kubeadm reset && init 過程中，留下了先前建立的舊網卡~

```bash
# cni0、flannel.1 刪除他，砍了他，踢了他
# 先移除 flannel
kubectl delete daemonsets -n kube-system -l app=flannel
sudo ip link delete cni.0
sudo ip link delete flannel.1
#
```

 重建 flannel 就好了，通體舒暢真舒服 XDDDD

```bash
# 舒服的服務狀態
[vagrant@kk8s-1 ~]$ kubectl get pod --all-namespaces --show-labels
NAMESPACE     NAME                             READY   STATUS    RESTARTS   AGE   LABELS
kube-system   coredns-576cbf47c7-dgktb         1/1     Running   0          24m   k8s-app=kube-dns,pod-template-hash=576cbf47c7
kube-system   coredns-576cbf47c7-vv7rc         1/1     Running   0          24m   k8s-app=kube-dns,pod-template-hash=576cbf47c7
kube-system   etcd-kk8s-1                      1/1     Running   1          90m   component=etcd,tier=control-plane
kube-system   kube-apiserver-kk8s-1            1/1     Running   2          90m   component=kube-apiserver,tier=control-plane
kube-system   kube-controller-manager-kk8s-1   1/1     Running   0          90m   component=kube-controller-manager,tier=control-plane
kube-system   kube-flannel-ds-amd64-cnsl2      1/1     Running   0          21m   app=flannel,controller-revision-hash=6697bf5fc6,pod-template-generation=1,tier=node
kube-system   kube-proxy-hrsdw                 1/1     Running   0          91m   controller-revision-hash=dc9c6bfb7,k8s-app=kube-proxy,pod-template-generation=1
kube-system   kube-scheduler-kk8s-1            1/1     Running   2          90m   component=kube-scheduler,tier=control-plane
[vagrant@kk8s-1 ~]$

[vagrant@kk8s-1 ~]$ kubectl get nodes
NAME     STATUS   ROLES    AGE   VERSION
kk8s-1   Ready    master   93m   v1.12.2
[vagrant@kk8s-1 ~]$
```

以上，過程中出現的問題，或許有解法直接新環境重建就好～  
但是，Lab 能夠發現問題，並找出問題根源，了解到故中原理才是我最開心最有收穫之處。

