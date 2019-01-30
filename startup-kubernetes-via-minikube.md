# Startup Local Kubernetes via Minikube

### 透過Minikube 創建本機 K8s環境

我先於自己Ubuntu電腦中，進行安裝 Minikube & Kubectl，去產生一個 K8s簡易環境。  
安裝步驟記錄如下：

{% hint style="info" %}
要使用Minikube，則主機上需具備Hypervisor環境，例如VirtualBox～
{% endhint %}

安裝參考頁：  
[https://kubernetes.io/docs/tasks/tools/install-minikube/](https://kubernetes.io/docs/tasks/tools/install-minikube/)  
[https://github.com/kubernetes/minikube/releases](https://github.com/kubernetes/minikube/releases)  
[https://kubernetes.io/docs/tasks/tools/install-kubectl/\#install-kubectl-binary-using-curl](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-binary-using-curl)

```bash
# 下載 minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \
  && sudo install minikube-linux-amd64 /usr/local/bin/minikube
minikube version
minikube version: v0.32.0

# 下載 kubectl
curl -Lo kubectl https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl \
&& chmod +x kubectl && sudo cp kubectl /usr/local/bin/ && rm kubectl

```

```bash
# 如僅安裝 minikube，執行時會提醒需安裝 kubectl。
[user@minikube ~]$ minikube status
========================================
kubectl could not be found on your path. kubectl is a requirement for using minikube
To install kubectl, please run the following:

curl -Lo kubectl https://storage.googleapis.com/kubernetes-release/release/v1.10.0/bin/linux/amd64/kubectl && chmod +x kubectl && sudo cp kubectl /usr/local/bin/ && rm kubectl

To disable this message, run the following:

minikube config set WantKubectlDownloadMsg false
========================================
minikube:
cluster:
kubectl:
[user@minikube ~]$
```

### 啟動 K8s環境

```bash
[user@minikube ~]$ minikube start
Starting local Kubernetes v1.12.4 cluster...
Starting VM...
Downloading Minikube ISO
 178.88 MB / 178.88 MB [============================================] 100.00% 0s
Getting VM IP address...
Moving files into cluster...
Setting up certs...
Connecting to cluster...
Setting up kubeconfig...
Stopping extra container runtimes...
Starting cluster components...
Verifying kubelet health ...
Verifying apiserver health ...Kubectl is now configured to use the cluster.
Loading cached images from config file.


Everything looks great. Please enjoy minikube!

```

#### 啟動前配置

```bash
# 啟動前，設定 minikube 佔用多少記憶體
minikube config set memory 4096

# 觀察 minikube 啟動時佔用多少記憶體
minikube config get memory
4096

minikube config view memory
- memory: 4096
```

### 觀察 K8s狀態

```bash
# Status
[user@minikube ~]$ minikube status
minikube: Running
cluster: Running
kubectl: Correctly Configured: pointing to minikube-vm at 192.168.99.100
[user@minikube ~]$

# Cluster Version
[user@minikube ~]$ kubectl version
Client Version: version.Info{Major:"1", Minor:"13", GitVersion:"v1.13.2", GitCommit:"cff46ab41ff0bb44d8584413b598ad8360ec1def", GitTreeState:"clean", BuildDate:"2019-01-10T23:35:51Z", GoVersion:"go1.11.4", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"12", GitVersion:"v1.12.4", GitCommit:"f49fa022dbe63faafd0da106ef7e05a29721d3f1", GitTreeState:"clean", BuildDate:"2018-12-14T06:59:37Z", GoVersion:"go1.10.4", Compiler:"gc", Platform:"linux/amd64"}
```

{% hint style="info" %}
在 kubectl version 資訊中看到兩個狀態：Client 指的是 Kubectl 執行檔本身、Server 指的是 K8s API 伺服器。
{% endhint %}

```bash
# Component Status
[user@minikube ~]$ kubectl get componentstatuses
NAME                 STATUS    MESSAGE              ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-0               Healthy   {"health": "true"}


# K8s node
[user@minikube ~]$ kubectl get node
NAME       STATUS   ROLES    AGE   VERSION
minikube   Ready    master   3m    v1.12.4


[user@minikube ~]$ kubectl get pods --all-namespaces
NAMESPACE     NAME                                    READY   STATUS    RESTARTS   AGE
kube-system   coredns-576cbf47c7-9gl6h                1/1     Running   0          4m31s
kube-system   coredns-576cbf47c7-rrgf4                1/1     Running   0          4m31s
kube-system   etcd-minikube                           1/1     Running   0          3m41s
kube-system   kube-addon-manager-minikube             1/1     Running   0          3m34s
kube-system   kube-apiserver-minikube                 1/1     Running   0          3m36s
kube-system   kube-controller-manager-minikube        1/1     Running   0          3m34s
kube-system   kube-proxy-8mzz2                        1/1     Running   0          4m31s
kube-system   kube-scheduler-minikube                 1/1     Running   0          3m34s
kube-system   kubernetes-dashboard-5bff5f8fb8-9dbng   1/1     Running   0          4m29s
kube-system   storage-provisioner                     1/1     Running   0          4m29s


[user@minikube ~]$ kubectl get services --all-namespaces
NAMESPACE     NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)         AGE
default       kubernetes             ClusterIP   10.96.0.1       <none>        443/TCP         6m18s
kube-system   kube-dns               ClusterIP   10.96.0.10      <none>        53/UDP,53/TCP   6m14s
kube-system   kubernetes-dashboard   ClusterIP   10.109.91.186   <none>        80/TCP          6m4s


[user@minikube ~]$ kubectl get deployments --all-namespaces
NAMESPACE     NAME                   DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kube-system   coredns                2         2         2            2           6m57s
kube-system   kubernetes-dashboard   1         1         1            1           6m47s


[user@minikube ~]$ kubectl get daemonsets --all-namespaces
NAMESPACE     NAME         DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
kube-system   kube-proxy   1         1         1       1            1           <none>          7m23s


[user@minikube ~]$ kubectl get replicasets --all-namespaces
NAMESPACE     NAME                              DESIRED   CURRENT   READY   AGE
kube-system   coredns-576cbf47c7                2         2         2       8m18s
kube-system   kubernetes-dashboard-5bff5f8fb8   1         1         1       8m16s
```

### 部署第一個 service / pod

部署參考頁：[https://kubernetes.io/docs/setup/minikube/\#installation](https://kubernetes.io/docs/setup/minikube/#installation)

```bash
# 部署
[user@minikube ~]$ kubectl run hello-minikube --image=k8s.gcr.io/echoserver:1.10 --port=8080
kubectl run --generator=deployment/apps.v1beta1 is DEPRECATED and will be removed in a future version. Use kubectl create instead.
deployment.apps/hello-minikube created
[user@minikube ~]$

[user@minikube ~]$ kubectl get pod
NAME                              READY   STATUS    RESTARTS   AGE
hello-minikube-7c77b68cff-l7smf   1/1     Running   0          26s

# Expose it as a new Kubernetes Services
[user@minikube ~]$ kubectl expose deployment hello-minikube --type=NodePort 
service/hello-minikube exposed

[user@minikube ~]$ kubectl get services
NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
hello-minikube   NodePort    10.97.245.120   <none>        8080:31944/TCP   9s
kubernetes       ClusterIP   10.96.0.1       <none>        443/TCP          4h
[user@minikube ~]$
```

### 測試 Service運行狀況

```bash
[user@minikube ~]$ curl $(minikube service hello-minikube --url)
Hostname: hello-minikube-7c77b68cff-l7smf

Pod Information:
	-no pod information available-

Server values:
	server_version=nginx: 1.13.3 - lua: 10008

Request Information:
	client_address=172.17.0.1
	method=GET
	real path=/
	query=
	request_version=1.1
	request_scheme=http
	request_uri=http://192.168.99.100:8080/

Request Headers:
	accept=*/*
	host=192.168.99.100:31944
	user-agent=curl/7.47.0

Request Body:
	-no body in request-

[user@minikube ~]$
```

### 刪除 Service / Pod

```bash
# Delete Services
[user@minikube ~]$ kubectl delete services hello-minikube
service "hello-minikube" deleted
# View Service \ Pod
[user@minikube ~]$ kubectl get service
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   4h

[user@minikube ~]$ kubectl get pod
NAME                              READY   STATUS    RESTARTS   AGE
hello-minikube-7c77b68cff-l7smf   1/1     Running   0          20m

# Delete Pod
[user@minikube ~]$ kubectl delete deployment hello-minikube
deployment.extensions "hello-minikube" deleted
# View Pod
[user@minikube ~]$ kubectl get pod
NAME                              READY   STATUS        RESTARTS   AGE
hello-minikube-7c77b68cff-l7smf   1/1     Terminating   0          20m
# 
[user@minikube ~]$ kubectl get pod
No resources found.


```

### 停止本機 K8s環境

```bash
[user@minikube ~]$ minikube stop
Stopping local Kubernetes cluster...
Machine stopped.

[user@minikube ~]$ minikube status
minikube: Stopped
cluster:
kubectl:
```

此篇，主要先自我練習本機 K8s環境從無到建立，然後再啟動第一個pod / service等所做的步驟記錄，下一篇再予以介紹上述牽涉到的觀念。  
感謝看官 :\)

