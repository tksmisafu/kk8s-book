# 繼上篇，排查 kubelet、kubeadm init 問題

### Fix failed to load Kubelet

在這篇 “[手工 Installing kubeadm](https://fufu.gitbook.io/kk8s/installing-kubeadm)” 文末，出現錯誤訊息：  
failed to load Kubelet config file /var/lib/kubelet/config.yaml  
是可以省略的，原因是在進行 kubeadm init 步驟階段，會自行產生。  
這是自己在進行 kubeadm init 步驟發現的情況。

### Fix kubeadm init

在這篇 “[手工 Installing CRI-O、kubeadm init](https://fufu.gitbook.io/kk8s/installing-crio-kubeadm-init)” 過程裡，kubeadm init 並沒有順利完成初始化目的  
先從 log 查看：

```bash
sudo journalctl -xeu kubelet
# 重點資訊：
RunPodSandbox from runtime service failed: rpc error: code = Unknown desc = cri-o configured with systemd cgroup manager

# kubeadm init log
This error is likely caused by:
	- The kubelet is not running
	- The kubelet is unhealthy due to a misconfiguration of the node in some way (required cgroups disabled)
```

在大多的官方、網友教學文章中都是以 docker 作為 Container Runtime Interface，  
但是我選擇 CRI-O，故在 cgroup 部分需要特別指定。  
預設是 docker cgroupfs，可以不需特別指定，CRI-O 需要指定 systemd。

```bash
# 查看 CRI-O 的 cgroup driver
[vagrant@kk8s-1 ~]$ cat /etc/crio/crio.conf | grep cgroup_manager
# cgroup_manager is the cgroup management implementation to be used
cgroup_manager = "systemd"
[vagrant@kk8s-1 ~]$

# 設定 cgroup-driver & ExecStart 增加 $KUBELET_CGROUP_ARGS
[vagrant@kk8s-1 ~]$ sudo vi /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
Environment="KUBELET_CGROUP_ARGS=--cgroup-driver=systemd"
ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS $KUBELET_CGROUP_ARGS

# 因 kubelet config file 有異動，必須進行 reload & restart kubelet
[vagrant@kk8s-1 ~]$ sudo systemctl daemon-reload
[vagrant@kk8s-1 ~]$ sudo systemctl restart kubelet

# 查看 kubelet status
[vagrant@kk8s-1 ~]$ systemctl status kubelet -l
● kubelet.service - kubelet: The Kubernetes Node Agent
   Loaded: loaded (/etc/systemd/system/kubelet.service; enabled; vendor preset: disabled)
  Drop-In: /etc/systemd/system/kubelet.service.d
           └─10-kubeadm.conf
   Active: active (running) since Sat 2018-11-10 17:12:16 UTC; 16m ago
     Docs: https://kubernetes.io/docs/
 Main PID: 514 (kubelet)
   CGroup: /system.slice/kubelet.service
           └─514 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --container-runtime=remote --container-runtime-endpoint=/var/run/crio/crio.sock --cgroup-driver=systemd

```

協助我找到問題的原因，除了上述 log 資訊，另外透過 Google 搜尋查看下列文章，補強我的觀念 [https://blog.csdn.net/zzq900503/article/details/81710319](https://blog.csdn.net/zzq900503/article/details/81710319)  
以上，解決了 kubeadm init 失敗的原因了

{% hint style="info" %}
kubelet config file：  
/etc/systemd/system/kubelet.service.d/10-kubeadm.conf
{% endhint %}

### 重新 kubeadm init

完成排除後，kubelet 確認運作正常，重新進行初始化動作～  
但是，不可馬上下 kubeadm init 指令，會偵測到目前相關環境已經 Ready，故須進行下面步驟：

```bash
[vagrant@kk8s-1 ~]$ sudo kubeadm reset
[vagrant@kk8s-1 ~]$ systemctl stop kubelet

# 發現 kube apiserver 運作中，需要 kill
[vagrant@kk8s-1 ~]$ ps aux |grep kube
[vagrant@kk8s-1 ~]$ sudo kill 31941 31965
```

重新進行 kubeadm init 指令，經過幾分鐘後，此時 K8s Master node 確認運作正常了

```bash
[vagrant@kk8s-1 ~]$ sudo kubeadm init --cri-socket="/var/run/crio/crio.sock" --apiserver-advertise-address=192.168.42.191
[init] using Kubernetes version: v1.12.2
[preflight] running pre-flight checks

...... <略過> ......

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

  kubeadm join 192.168.42.191:6443 --token sll2r4.z7dlw7e2yt1e8b3p --discovery-token-ca-cert-hash sha256:c06f9142687d0b34871aa1e3a2a6dcbfe0edf752bed5e7891a77e2d4fcc60dac

[vagrant@kk8s-1 ~]$

```

#### 後續

```bash
# 執行本機帳號配置設定  
[vagrant@kk8s-1 ~]$ mkdir -p $HOME/.kube
[vagrant@kk8s-1 ~]$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
[vagrant@kk8s-1 ~]$ sudo chown $(id -u):$(id -g) $HOME/.kube/config

# 檢查 K8s node status
[vagrant@kk8s-1 ~]$ kubectl get nodes
NAME     STATUS   ROLES    AGE   VERSION
kk8s-1   Ready    master   16m   v1.12.2

```

#### 很高興完成自己環境中的 kubeadm init 里程碑 ＾o＾

