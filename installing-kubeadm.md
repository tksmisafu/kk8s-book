# 手工 Installing kubeadm

### 環境準備

在自己 Lab host 中，透過 Vagrant 方式建立三個 VM 資源進行模擬。

首先，建立三個 VM 目錄，並且初始化及啟動 VM，三 VM 環境依序下列相同步驟。

```bash
[afu@Lab ~/vv]$ mkdir kk8s-1
[afu@Lab ~/vv]$ mkdir kk8s-2
[afu@Lab ~/vv]$ mkdir kk8s-3
[afu@Lab ~/vv]$ cd kk8s-3
# Vagrant 初始化
[afu@Lab ~/vv/kk8s-1]$ vagrant init centos/7
# 確認 VM 資源：IP、CPU、RAM
[afu@Lab ~/vv/kk8s-1]$ vi Vagrantfile
#
  config.vm.network "public_network",  ip: "192.168.42.191"
  config.vm.provider "virtualbox" do |vb|
      vb.memory = "4096"
      vb.cpus = 4
  end
#
啟動 VM
[afu@Lab ~/vv/kk8s-1]$ vagrant up
登入 VM
[afu@Lab ~/vv/kk8s-1]$ vagrant ssh

```

### 環境需求

```bash
# for CentOS
# 設定 hostname
sudo hostname -b kk8s-1

# 1. 關閉 SWAP
[vagrant@kk8s-1 ~]$ sudo swapoff -a
[vagrant@kk8s-1 ~]$ sudo vi /etc/fstab
#
/dev/mapper/VolGroup00-LogVol00 /                       xfs     defaults        0 0
UUID=570897ca-e759-4c81-90cf-389da6eee4cc /boot                   xfs     defaults        0 0
#/dev/mapper/VolGroup00-LogVol01 swap                    swap    defaults        0 0

# 2. Set SELinux in permissive mode (effectively disabling it)
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

# 3. Some users on RHEL/CentOS 7 have reported issues with traffic being routed incorrectly due to iptables being bypassed.
# You should ensure net.bridge.bridge-nf-call-iptables is set to 1 in your sysctl config,
# e.g.
[vagrant@kk8s-1 ~]$ sudo vi /etc/sysctl.d/k8s.conf
    net.bridge.bridge-nf-call-ip6tables = 1
    net.bridge.bridge-nf-call-iptables = 1
    vm.swappiness=0
    
[vagrant@kk8s-1 ~]$ sudo sysctl --system
* Applying /usr/lib/sysctl.d/00-system.conf ...
* Applying /usr/lib/sysctl.d/10-default-yama-scope.conf ...
kernel.yama.ptrace_scope = 0
* Applying /usr/lib/sysctl.d/50-default.conf ...
kernel.sysrq = 16
kernel.core_uses_pid = 1
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.accept_source_route = 0
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.default.promote_secondaries = 1
net.ipv4.conf.all.promote_secondaries = 1
fs.protected_hardlinks = 1
fs.protected_symlinks = 1
* Applying /etc/sysctl.d/99-sysctl.conf ...
* Applying /etc/sysctl.d/k8s.conf ...
* Applying /etc/sysctl.conf ...
[vagrant@kk8s-1 ~]$
```

### Installing kubeadm, kubelet and kubectl <a id="installing-kubeadm-kubelet-and-kubectl"></a>

```bash
# 新增 repo
sudo vi /etc/yum.repos.d/kubernetes.repo
#
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kube*
#

# 安裝 kubelet kubeadm kubectl
sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

# 確認套件版本資訊
[vagrant@kk8s-1 ~]$ kube
kubeadm  kubectl  kubelet

[vagrant@kk8s-1 ~]$ kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"12", GitVersion:"v1.12.2", GitCommit:"17c77c7898218073f14c8d573582e8d2313dc740", GitTreeState:"clean", BuildDate:"2018-10-24T06:51:33Z", GoVersion:"go1.10.4", Compiler:"gc", Platform:"linux/amd64"}

[vagrant@kk8s-1 ~]$ kubectl version
Client Version: version.Info{Major:"1", Minor:"12", GitVersion:"v1.12.2", GitCommit:"17c77c7898218073f14c8d573582e8d2313dc740", GitTreeState:"clean", BuildDate:"2018-10-24T06:54:59Z", GoVersion:"go1.10.4", Compiler:"gc", Platform:"linux/amd64"}
The connection to the server localhost:8080 was refused - did you specify the right host or port?

[vagrant@kk8s-1 ~]$ kubelet --version
Kubernetes v1.12.2
```

### 啟動 kubelet

```bash
[vagrant@kk8s-1 ~]$ sudo systemctl enable kubelet && sudo systemctl start kubelet
Created symlink from /etc/systemd/system/multi-user.target.wants/kubelet.service to /etc/systemd/system/kubelet.service.

[vagrant@kk8s-1 ~]$ sudo systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
   Loaded: loaded (/etc/systemd/system/kubelet.service; enabled; vendor preset: disabled)
  Drop-In: /etc/systemd/system/kubelet.service.d
           └─10-kubeadm.conf
   Active: activating (auto-restart) (Result: exit-code) since Fri 2018-11-09 14:27:15 UTC; 551ms ago
     Docs: https://kubernetes.io/docs/
  Process: 3580 ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS (code=exited, status=255)
 Main PID: 3580 (code=exited, status=255)
```

### 啟動 kubelet 失敗

```bash
# 從下面 log 發現，缺少了一個配置檔案
# /var/lib/kubelet/config.yaml
[vagrant@kk8s-1 ~]$ sudo tailf /var/log/messages
Nov  9 14:26:13 localhost systemd: kubelet.service holdoff time over, scheduling restart.
Nov  9 14:26:13 localhost systemd: Started kubelet: The Kubernetes Node Agent.
Nov  9 14:26:13 localhost systemd: Starting kubelet: The Kubernetes Node Agent...
Nov  9 14:26:13 localhost kubelet: F1109 14:26:13.714784    3515 server.go:190] failed to load Kubelet config file /var/lib/kubelet/config.yaml, error failed to read kubelet config file "/var/lib/kubelet/config.yaml", error: open /var/lib/kubelet/config.yaml: no such file or directory
Nov  9 14:26:13 localhost systemd: kubelet.service: main process exited, code=exited, status=255/n/a
Nov  9 14:26:13 localhost systemd: Unit kubelet.service entered failed state.
Nov  9 14:26:13 localhost systemd: kubelet.service failed.

# 問題先留著，下一篇會 fix
```

{% hint style="info" %}
文章出處：  
[https://kubernetes.io/docs/setup/independent/install-kubeadm/](https://kubernetes.io/docs/setup/independent/install-kubeadm/)
{% endhint %}

下一篇，準備往 [Using kubeadm to Create a Cluster](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/) 邁進。

