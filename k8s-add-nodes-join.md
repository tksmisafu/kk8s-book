# K8s add Nodes（join）

### 環境準備

我準備了兩個 VM 當 K8s work node，加入我的 K8s Lab 叢集環境。  
先進行 kubeadm 套件安裝：  
安裝 kubeadm：[https://fufu.gitbook.io/kk8s/installing-kubeadm\#jing-xu-qiu](https://fufu.gitbook.io/kk8s/installing-kubeadm#jing-xu-qiu)  
安裝 CRI-O：[https://fufu.gitbook.io/kk8s/installing-crio-kubeadm-init\#install-cri-o](https://fufu.gitbook.io/kk8s/installing-crio-kubeadm-init#install-cri-o) 

因採用非預設的 CRI，故需指定 cgroup-driver

```bash
[vagrant@kk8s-1 ~]$ sudo vi /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
Environment="KUBELET_CGROUP_ARGS=--cgroup-driver=systemd"
ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS $KUBELET_CGROUP_ARGS

```

### 進行 node join

過程中遇到兩個問題，記錄下來：

#### 錯誤1：IPVS proxier will not be used

```bash
[WARNING RequiredIPVSKernelModulesAvailable]: 
the IPVS proxier will not be used, because the following required kernel modules are not loaded: 
[ip_vs ip_vs_rr ip_vs_wrr ip_vs_sh] or no builtin kernel ipvs support: 
map[ip_vs:{} ip_vs_rr:{} ip_vs_wrr:{} ip_vs_sh:{} nf_conntrack_ipv4:{}]

you can solve this problem with following methods:
  1. Run 'modprobe -- ' to load missing kernel modules;
  2. Provide the missing builtin kernel ipvs support
```

經過 Google 搜尋網友經驗，需要下這些指令：

```bash
sudo modprobe ip_vs_rr
sudo modprobe ip_vs_wrr
sudo modprobe ip_vs_sh
sudo modprobe ip_vs
sudo modprobe nf_conntrack_ipv4

# 檢查：
sudo lsmod| grep ip_vs
```

#### 錯誤2：ERROR FileAvailable

```bash
[preflight] Some fatal errors occurred:
	[ERROR FileAvailable--etc-kubernetes-kubelet.conf]: /etc/kubernetes/kubelet.conf already exists
	[ERROR FileAvailable--etc-kubernetes-bootstrap-kubelet.conf]: /etc/kubernetes/bootstrap-kubelet.conf already exists
	[ERROR FileAvailable--etc-kubernetes-pki-ca.crt]: /etc/kubernetes/pki/ca.crt already exists
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
```

這是因為我已經執行數次 join 指令，故已經產生過上述檔案。  
透過`--ignore-preflight-errors=all`即可忽略！

#### Join 指令

```bash
sudo kubeadm join 192.168.42.191:6443 --token noduzf.t2ng9nhcx2gdo53y \
--discovery-token-ca-cert-hash sha256:d02f588e50c1f16fb9eec512e5ca14fd2add98e0231805a46c4e0fb2aff83555 \
--cri-socket="/var/run/crio/crio.sock" \
--ignore-preflight-errors=all
# 成功完成 join
```

#### 錯誤3：connection to the server localhost:8080 was refused

```bash
[vagrant@kk8s-2 ~]$ kubectl get nodes
The connection to the server localhost:8080 was refused - did you specify the right host or port?

```

這是是因為存取不到 API server，需設定本機帳號 API 存取權限

```bash
mkdir -p $HOME/.kube
scp master-ip:$HOME/.kube/config $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

[vagrant@kk8s-2 ~]$ kubectl get nodes
NAME     STATUS   ROLES    AGE   VERSION
kk8s-1   Ready    master   23h   v1.12.2
kk8s-2   Ready    <none>   24m   v1.12.2
kk8s-3   Ready    <none>   17m   v1.12.2
```

#### 終於完成另一個小里程～經驗與概念又學習到了～

