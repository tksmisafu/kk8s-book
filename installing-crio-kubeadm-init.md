# 手工 Installing CRI-O、kubeadm init

上一篇，其實少裝了一個重要角色 Container Runtime Interface，簡稱 CRI  
這一篇，來裝吧～

### CRI-O

我選擇 Container Runtime 是 CRI-O，是個專為 K8s 而設計，並且完全支持符合 OCI 規範的任何容器。  
像是 Redhat 商用 K8s 平台：OpenShift 即使以 CRI-O 作為預設的 Container Runtime。  
開源的 CRI 專案網址：[https://github.com/kubernetes-sigs/cri-o](https://github.com/kubernetes-sigs/cri-o)  
官網網址：[http://cri-o.io/](http://cri-o.io/)

![](.gitbook/assets/cri-o.svg)

### CRI-O  Compatibility &lt;-&gt; Kubernetes clusters

| Version - Branch | Kubernetes branch/version | Maintenance status |
| :--- | :--- | :--- |
| CRI-O 1.0.x - release-1.0 | Kubernetes 1.7 branch, v1.7.x | ＝ |
| CRI-O 1.8.x - release-1.8 | Kubernetes 1.8 branch, v1.8.x | ＝ |
| CRI-O 1.9.x - release-1.9 | Kubernetes 1.9 branch, v1.9.x | ＝ |
| CRI-O 1.10.x - release-1.10 | Kubernetes 1.10 branch, v1.10.x | ＝ |
| CRI-O 1.11.x - release-1.11 | Kubernetes 1.11 branch, v1.11.x | ＝ |
| CRI-O 1.12.x - release-1.12 | Kubernetes 1.12 branch, v1.12.x | ＝ |
| CRI-O HEAD - master | Kubernetes master branch | ✓ |

* Key:
  * `✓` Changes in main Kubernetes repo about CRI are actively implemented in CRI-O
  * `=` Maintenance is manual, only bugs will be patched.

### K8s CRI-O Architecture

![](.gitbook/assets/cri-o-architecture.png)

### Install CRI-O

#### Prerequisites <a id="prerequisites"></a>

```bash
modprobe overlay
modprobe br_netfilter

# Setup required sysctl params, these persist across reboots.
[vagrant@kk8s-1 ~]$ sudo vi /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1

[vagrant@kk8s-1 ~]$ sudo sysctl --system
* Applying /etc/sysctl.d/k8s.conf ...
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
vm.swappiness = 0
* Applying /etc/sysctl.conf ...
```

#### Install prerequisites & CRI-O

```bash
# Install prerequisites
[vagrant@kk8s-1 ~]$ sudo yum-config-manager --add-repo=https://cbs.centos.org/repos/paas7-crio-311-candidate/x86_64/os/

# Install CRI-O
[vagrant@kk8s-1 ~]$ sudo yum install --nogpgcheck cri-o
#
Installed:
  cri-o.x86_64 0:1.11.8-2.rhaos3.11.git71cc465.el7

Dependency Installed:
  audit-libs-python.x86_64 0:2.8.1-3.el7_5.1              checkpolicy.x86_64 0:2.5-6.el7                                       container-selinux.noarch 2:2.68-1.el7
  containernetworking-plugins.x86_64 0:0.7.1-1.el7        criu.x86_64 0:3.5-4.el7                                              libcgroup.x86_64 0:0.41-15.el7
  libnet.x86_64 0:1.1.6-7.el7                             libsemanage-python.x86_64 0:2.5-11.el7                               policycoreutils-python.x86_64 0:2.5-22.el7
  protobuf-c.x86_64 0:1.0.2-3.el7                         python-IPy.noarch 0:0.75-6.el7                                       runc.x86_64 0:1.0.0-52.dev.git70ca035.el7_5
  setools-libs.x86_64 0:3.3.8-2.el7                       skopeo-containers.x86_64 1:0.1.31-1.dev.gitae64ff7.el7.centos

Dependency Updated:
  audit.x86_64 0:2.8.1-3.el7_5.1                                                        audit-libs.x86_64 0:2.8.1-3.el7_5.1

```

#### Start CRI-O

```bash
[vagrant@kk8s-1 ~]$ sudo systemctl start crio
[vagrant@kk8s-1 ~]$
[vagrant@kk8s-1 ~]$ sudo systemctl status crio
● crio.service - Open Container Initiative Daemon
   Loaded: loaded (/usr/lib/systemd/system/crio.service; disabled; vendor preset: disabled)
   Active: active (running) since Sat 2018-11-10 11:52:18 UTC; 3s ago
     Docs: https://github.com/kubernetes-sigs/cri-o
 Main PID: 16765 (crio)
   CGroup: /system.slice/crio.service
           └─16765 /usr/bin/crio

Nov 10 11:52:18 kk8s-1 systemd[1]: Starting Open Container Initiative Daemon...
Nov 10 11:52:18 kk8s-1 crio[16765]: time="2018-11-10 11:52:18.905802494Z" level=error msg="watcher.Add("/usr/share/containers/oci/hooks.d") failed: no such file or directory"
Nov 10 11:52:18 kk8s-1 systemd[1]: Started Open Container Initiative Daemon.

# 上述出現一個 error，透過 Google 搜尋後發現需要 hook 目錄，重啟 crio 即可。
# https://github.com/containers/libpod/blob/master/pkg/hooks/docs/oci-hooks.5.md
[vagrant@kk8s-1 ~]$ sudo mkdir /usr/share/containers/oci/
[vagrant@kk8s-1 ~]$ sudo mkdir /usr/share/containers/oci/hooks.d
[vagrant@kk8s-1 ~]$ sudo systemctl restart crio
[vagrant@kk8s-1 ~]$ crio --version
crio version 1.11.8
```

### kubeadm init

```bash
# 因 Lab 環境有兩個網路介面，為了確定 kubeadm 抓取正確的 IP 參數，透過 hosts 確認主機名稱解析的 IP。
sudo vi /etc/hosts

# 初始化叢集 Master node
sudo kubeadm init --apiserver-advertise-address=192.168.42.191
# 因預設 kubeadm init 會偵測環境中 docker 要素但我沒安裝 docker！
# [preflight] WARNING: Couldn't create the interface used for talking to the container runtime: docker is required for container runtime: exec: "docker": executable file not found in $PATH
# 我採用 CRI-O 作為 Container Runtime，故 init arg 需增加 --cri-socket，如下：
sudo kubeadm init --cri-socket="/var/run/crio/crio.sock" --apiserver-advertise-address=192.168.42.191

### 結果失敗，主因出在 kubelet 服務上～只好繼續排查問題。
# 下篇繼續～
```

{% hint style="danger" %}
This error is likely caused by:

* The kubelet is not running
* The kubelet is unhealthy due to a misconfiguration of the node in some way \(required cgroups disabled\)
{% endhint %}

{% hint style="info" %}
文章出處：  
[https://kubernetes.io/docs/setup/independent/install-kubeadm/\#installing-runtime](https://kubernetes.io/docs/setup/independent/install-kubeadm/#installing-runtime)  
[https://kubernetes.io/docs/setup/cri/\#prerequisites](https://kubernetes.io/docs/setup/cri/#prerequisites)
{% endhint %}



