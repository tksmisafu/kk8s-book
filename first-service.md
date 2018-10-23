# 回顧第一次部署，淺談 Service

[上一篇](https://fufu.gitbook.io/kk8s/first-pod-deployment)，提到 deployment 物件，此物件與建立 service 物件有連帶關係的～  
透過回顧[此篇](https://fufu.gitbook.io/kk8s/startup-kubernetes-via-minikube)，可以知道透過`kubectl export`指令，即會建立 service 物件。

```bash
# Expose it as a new Kubernetes Services
[user@minikube ~]$ kubectl expose deployment hello-minikube --type=NodePort service/hello-minikube exposed

```

### 查看 Service 資訊

```bash
[user@minikube ~]$ kubectl get services -o wide
NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE   SELECTOR
hello-minikube   NodePort    10.97.219.147   <none>        8080:31472/TCP   3d    run=hello-minikube
kubernetes       ClusterIP   10.96.0.1       <none>        443/TCP          5d    <none>
[user@minikube ~]$

[user@minikube ~]$ kubectl describe services hello-minikube
Name:                     hello-minikube
Namespace:                default
Labels:                   run=hello-minikube
Annotations:              <none>
Selector:                 run=hello-minikube
Type:                     NodePort
IP:                       10.97.219.147
Port:                     <unset>  8080/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  31472/TCP
Endpoints:                172.17.0.4:8080
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>

```

上面的資訊中～透過 kubectl get services 會看見兩個 service～   
其中，kubernetes 是由 K8s 叢集自動建立的，其用途就是讓應用程式與 API 溝通。  
看下項目 hello-minikube service 物件：

* 欄位 SELECTOR 中可以看到 “run=hello-minikube” 資訊，這是由 deployment 物件中 Selector 定義並提取出來。
* 欄位 PORT\(S\) 看到 8080 埠，這是由 deployment 物件中 Port 定義並提取出來。
* 10.97.219.147 此 IP 是虛擬 IP，存在於 pod 內的網路資訊。

{% hint style="info" %}
參考書籍：   
**Kubernetes 建置與執行** 書中 P.78~79 內容
{% endhint %}

### 開放 Service 給予外部存取

在預設情況下，pod 物件僅能於叢集之間存取，但例如此篇案例 hello-minikube 是個 http 應用程式  
很多場景下，HTTP 應用層需要開放“埠”端口讓外部存取。

* 有個簡單方式是使用`--type=NodePort`方式，透過 service 讓 K8s node 隨機監聽一個埠端口。
  * 讓你無需關注 pod 運作在哪個 K8s node。
  * `NodePort:  <unset>  31472/TCP` 就是隨機產生的“埠”端口。

{% hint style="info" %}
參考書籍：   
**Kubernetes 建置與執行** 書中 P.83 內容
{% endhint %}

