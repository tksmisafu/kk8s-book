# 回顧第一次部署，淺談 Pod、Deployment

#### 我在[此篇文章](https://fufu.gitbook.io/kk8s/startup-kubernetes-via-minikube#bu-shu-di-yi-service-pod)進行了第一次 Container 部署，以此範例淺談 Pod、Deployments 基本概念。

### 淺談 Pod

* 當要進行 Pods 服務部署，透過執行`kubectl run <name> --image=<soruce> --port=<number>`，K8s 將建立取名為 &lt;name&gt; 之 pod，並且同時建立 deployment 物件。
* 預設下，`kubectl get`會獲得簡化後得資訊，另外可透過`-o wide`或者`kubectl describe`取得更詳盡得物件資訊。
* 回顧服務部署，並透過 _**物件查詢指令**_ 可查出相關物件資訊：

```bash
# <hello-minikube> 服務部署
[user@minikube ~]$ kubectl run hello-minikube --image=k8s.gcr.io/echoserver:1.10 --port=8080
kubectl run --generator=deployment/apps.v1beta1 is DEPRECATED and will be removed in a future version. Use kubectl create instead.
deployment.apps/hello-minikube created

######################################
#        以下是“物件查詢指令”           #
######################################
# 查詢運行中的 pod，出現 <hello-minikube-id>
# 內容有 POD-NAME、READY_數量狀態、STATUS_運作狀態、RESTARTS_重啟次數、AGE_生命周長
[user@minikube ~]$ kubectl get pod
NAME                              READY   STATUS    RESTARTS   AGE
hello-minikube-7c77b68cff-ncqmn   1/1     Running   1          3d

[user@minikube ~]$ kubectl get pod -o wide
NAME                              READY   STATUS    RESTARTS   AGE   IP           NODE
hello-minikube-7c77b68cff-ncqmn   1/1     Running   1          3d    172.17.0.4   minikube


######################################
# 在此先查詢 service，尚無 <hello-minikube>，後續篇幅會為此介紹。
[user@minikube ~]$ kubectl get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   3d

```

#### `kubectl describe`取得更多資訊，可分為三大項：

* Pod 基本資訊
* Container 詳細資訊
* Events 事件紀錄

```bash
######################################
# 查看更多 pod 的詳細資訊
[user@minikube ~]$ kubectl describe pods hello-minikube
## 以下是 Pod 資訊。
Name:           hello-minikube-7c77b68cff-ncqmn
Namespace:      default
Node:           minikube/10.0.2.15
Start Time:     Fri, 19 Oct 2018 00:59:19 +0800
Labels:         pod-template-hash=3733624799
                run=hello-minikube
Annotations:    <none>
Status:         Running
IP:             172.17.0.4
Controlled By:  ReplicaSet/hello-minikube-7c77b68cff
## 以下是 Container 資訊。
Containers:
  hello-minikube:
    Container ID:   docker://1b32f7f5e403f49365fc071a541e28e0a7eddcfe9d4fc2aa33044cefe2a0ebde
    Image:          k8s.gcr.io/echoserver:1.10
    Image ID:       docker-pullable://k8s.gcr.io/echoserver@sha256:cb5c1bddd1b5665e1867a7fa1b5fa843a47ee433bbb75d4293888b71def53229
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Fri, 19 Oct 2018 18:03:10 +0800
    Last State:     Terminated
      Reason:       Error
      Exit Code:    255
      Started:      Fri, 19 Oct 2018 00:59:20 +0800
      Finished:     Fri, 19 Oct 2018 18:02:44 +0800
    Ready:          True
    Restart Count:  1
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-qfxf5 (ro)
Conditions:
  Type           Status
  Initialized    True
  Ready          True
  PodScheduled   True
Volumes:
  default-token-qfxf5:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-qfxf5
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
## 以下是 Events 資訊。
Events:          <none>

[user@minikube ~]$
```

{% hint style="info" %}
參考書籍：  
**Kubernetes 建置與執行** 書中 P.46~50 內容
{% endhint %}

### 淺談 Deployment

經過 kubectl run 完成服務部署後，同時間也會建立 deployment 物件。

* deployment 物件，是用於管理新版本的發佈，經常有 rollout 動作執行。
* deployment 物件也管理著 ReplicaSet 物件。
  * 正因如此，不需再透過 replicaset 物件進行 scale 動作，除非先移除了 deployment 物件關係，但非常不建議這麼做。
* 因此未來可透過 deployment 物件對此 pod 進行宣告式動作，例如更版、擴展、Rollout 等等。
* `kubectl describe deployment`兩項重要資訊：
  * OldReplicaSets
  * NewReplicaSet
  * 這兩項代表著 deployments 所管理得 replicaset 物件。
  * 如果 rollout 進行中，則兩個欄位都有資訊。
  * 如果 rollout 完成了，則 OldReplicaSets 會顯示 &lt;none&gt; 資訊。

```bash
######################################
# 查詢 deployment，出現 <hello-minikube> deployment 物件
[user@minikube ~]$ kubectl get deployments
NAME             DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
hello-minikube   1         1         1            1           3d


[user@minikube ~]$ kubectl get deployments -o wide
NAME             DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS       IMAGES                       SELECTOR
hello-minikube   1         1         1            1           3d    hello-minikube   k8s.gcr.io/echoserver:1.10   run=hello-minikube


######################################
# 查看更多 deployment 的詳細資訊
[user@minikube ~]$ kubectl describe deployments hello-minikube
Name:                   hello-minikube
Namespace:              default
CreationTimestamp:      Fri, 19 Oct 2018 00:59:19 +0800
Labels:                 run=hello-minikube
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               run=hello-minikube
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  run=hello-minikube
  Containers:
   hello-minikube:
    Image:        k8s.gcr.io/echoserver:1.10
    Port:         8080/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   hello-minikube-7c77b68cff (1/1 replicas created)
Events:          <none>
[user@minikube ~]$
```

{% hint style="info" %}
參考書籍：  
**Kubernetes 建置與執行** 書中 P.150 ~ 152、P.154 內容
{% endhint %}



