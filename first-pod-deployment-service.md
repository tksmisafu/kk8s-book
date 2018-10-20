# 回顧第一次部署，淺談 Pod、Deployments、Service

#### 我在[此篇文章](https://fufu.gitbook.io/kk8s/startup-kubernetes-via-minikube#bu-shu-di-yi-service-pod)進行了第一次 Container 部署，以此範例淺談 Pod、Deployments、Service 基本概念。

* 當要進行 Container 服務部署，透過執行`kubectl run <name> --image=<soruce> --port=<number>`，K8s 將建立取名為 &lt;name&gt; 之 pod，並且同時建立 deployments 物件。
* 承上述，因此後續可對此 pod 進行宣告式動作，例如更版、擴展、Rollout 等等。
* 回顧服務部署，並透過物件查詢指令可查出相關物件資訊：

```bash
# <hello-minikube> 服務部署
[user@minikube ~]$ kubectl run hello-minikube --image=k8s.gcr.io/echoserver:1.10 --port=8080
kubectl run --generator=deployment/apps.v1beta1 is DEPRECATED and will be removed in a future version. Use kubectl create instead.
deployment.apps/hello-minikube created

######################################
# 查詢運行中的 pod，出現 <hello-minikube-id>
# 內容有 POD-NAME、READY_數量狀態、STATUS_運作狀態、RESTARTS_重啟次數、AGE_生命周長
[user@minikube ~]$ kubectl get pod
NAME                              READY   STATUS    RESTARTS   AGE
hello-minikube-7c77b68cff-ncqmn   1/1     Running   0          1m

######################################
# 查詢 deployments，出現 <hello-minikube> deployments 物件
[user@minikube ~]$ kubectl get deployments
NAME             DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
hello-minikube   1         1         1            1           1m

######################################
# 查詢 service，尚無 <hello-minikube>
[user@minikube ~]$ kubectl get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   1d

######################################
# 了解更多 pod 的詳細資訊
[user@minikube ~]$ kubectl describe pods hello-minikube

######################################
# 了解更多 deployment 的詳細資訊
[user@minikube ~]$ kubectl describe deployments hello-minikube

```

{% hint style="info" %}
參考書籍：  
**Kubernetes 建置與執行** 書中 P.46~50、P.150、P.154 內容
{% endhint %}

#### ㄎaasdd

```bash
####################################################################################

# Expose it as a new Kubernetes Service
[user@minikube ~]$ kubectl expose deployment hello-minikube --type=NodePort
service/hello-minikube exposed
######################################
[user@minikube ~]$ kubectl get pod
NAME                              READY   STATUS    RESTARTS   AGE
hello-minikube-7c77b68cff-ncqmn   1/1     Running   0          4m
######################################
[user@minikube ~]$ kubectl get services
NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
hello-minikube   NodePort    10.97.219.147   <none>        8080:31472/TCP   32s
kubernetes       ClusterIP   10.96.0.1       <none>        443/TCP          1d
######################################
[user@minikube ~]$ kubectl get deployments
NAME             DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
hello-minikube   1         1         1            1           5m
######################################

```

