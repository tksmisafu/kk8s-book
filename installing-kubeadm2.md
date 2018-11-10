# 手工 Installing kubeadm-2

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



{% hint style="info" %}
文章出處：  
[https://kubernetes.io/docs/setup/cri/\#prerequisites](https://kubernetes.io/docs/setup/cri/#prerequisites)
{% endhint %}

