# Pod 副本管理～ 實作篇

### 觀察現況

我們透過 ReplicaSet 物件提交到 K8s API 的方式建立副本管理。  
先行察看兩個物件，可以得知哪一個 pod 尚未套用 ReplicaSet 物件～  
`kubectl get pods`  
`kubectl get rs`

```bash
[user@minikube ~]$ kubectl get pods
NAME                              READY   STATUS    RESTARTS   AGE
hello-kitty                       1/1     Running   0          6d
hello-minikube-7c77b68cff-ncqmn   1/1     Running   1          11d
hello-moto-6456dff9f-67jh5        1/1     Running   0          5d

[user@minikube ~]$ kubectl get rs
NAME                        DESIRED   CURRENT   READY   AGE
hello-minikube-7c77b68cff   1         1         1       11d
hello-moto-6456dff9f        1         1         1       5d
```

另一個方式查看該 pod 是否已有 ReplicaSet 管理  
`kubectl get pods -o yaml`  
如果顯示的內容中，有出現`kind: ReplicaSet`物件，即代表受管理中

```bash
  ...
  ownerReferences:
  - apiVersion: extensions/v1beta1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: hello-moto-6456dff9f

```

### 新增 ReplicaSet 實例

經比對，pod **hello-kitty** 尚未套用 ReplicaSet，可以最為下列範例～   
新增 **hello-kitty** 的 ReplicaSet 物件～`rs-hello-kitty.yaml`

```yaml
apiVersion: extensions/v1beta1
kind: ReplicaSet
metadata:
  name: hello-kitty
spec:
  replicas: 2
  template:
    metadata:
      labels:
        run: hello-kitty
    spec:
      containers:
        - name: hello-kitty
          image: "k8s.gcr.io/echoserver:1.10"
```

套用 ReplicaSet 物件之前 Pod 狀態：

```bash
[user@minikube ~]$ kubectl get pods hello-kitty -o wide
NAME          READY   STATUS    RESTARTS   AGE   IP           NODE
hello-kitty   1/1     Running   0          5d    172.17.0.6   minikube

```

套用 ReplicaSet 物件：

```bash
[user@minikube ~]$ kubectl apply -f kk8s/rs-hello-kitty.yaml
replicaset.extensions/hello-kitty created
```

套用 ReplicaSet 物件之後 Pod 狀態：

```bash
[user@minikube ~]$ kubectl get pods
NAME                              READY   STATUS    RESTARTS   AGE
hello-kitty                       1/1     Running   0          6d
hello-kitty-lk7lg                 1/1     Running   0          1m
hello-kitty-x94s2                 1/1     Running   0          1m
hello-minikube-7c77b68cff-ncqmn   1/1     Running   1          11d
hello-moto-6456dff9f-67jh5        1/1     Running   0          5d

```

檢查 ReplicaSet，可以看見裏頭的 Labels、replicas 資訊：

```bash
[user@minikube ~]$ kubectl describe rs hello-kitty
Name:         hello-kitty
Namespace:    default
Selector:     run=hello-kitty
Labels:       run=hello-kitty
Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"extensions/v1beta1","kind":"ReplicaSet","metadata":{"annotations":{},"name":"hello-kitty","namespace":"default"},"spec":{"r...
Replicas:     2 current / 2 desired
Pods Status:  2 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  run=hello-kitty
  Containers:
   hello-kitty:
    Image:        k8s.gcr.io/echoserver:1.10
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age   From                   Message
  ----    ------            ----  ----                   -------
  Normal  SuccessfulCreate  84s   replicaset-controller  Created pod: hello-kitty-lk7lg
  Normal  SuccessfulCreate  84s   replicaset-controller  Created pod: hello-kitty-x94s2

```

### Scale Pod 方式一

進行擴展 Pod`[2 --> 4]`，透過指令`kubectl scale`

```bash
[user@minikube ~]$ kubectl scale --replicas=4 rs/hello-kitty
replicaset.extensions/hello-kitty scaled

[user@minikube ~]$ kubectl get rs hello-kitty
NAME          DESIRED   CURRENT   READY   AGE
hello-kitty   4         4         4       9h

[user@minikube ~]$ kubectl get pods
NAME                              READY   STATUS    RESTARTS   AGE
hello-kitty                       1/1     Running   0          6d
hello-kitty-d7tfq                 1/1     Running   0          22s
hello-kitty-lk7lg                 1/1     Running   0          9h
hello-kitty-x94s2                 1/1     Running   0          9h
hello-kitty-zgrb8                 1/1     Running   0          22s
hello-minikube-7c77b68cff-ncqmn   1/1     Running   1          11d
hello-moto-6456dff9f-67jh5        1/1     Running   0          6d

```

{% hint style="info" %}
書中提到上述指令：`kubectl scale <rs-name> --replicas=value，因為 K8s 迭代，透過 kubectl scale -h 得知語法已經變更。`
{% endhint %}

### Scale Pod 方式二

透過 kubectl apply 進行宣告式scale`[4 --> 3]`  
編輯 rs-hello-kitty.yaml 內容中的 replicas: value 進行管理

```yaml
apiVersion: extensions/v1beta1
kind: ReplicaSet
metadata:
  name: hello-kitty
spec:
  replicas: 3  # 修改此行 value
```

```bash
# 套用新 replicas value 設定
[user@minikube ~]$ kubectl apply -f kk8s/rs-hello-kitty.yaml
replicaset.extensions/hello-kitty configured

# 確認已經生效，從剛剛稍早 pod hello-kitty 4個，變成 3個。
[user@minikube ~]$ kubectl get rs hello-kitty
NAME          DESIRED   CURRENT   READY   AGE
hello-kitty   3         3         3       10h

# 觀察 pod 變化，副本 hello-kitty-d7tfq 正在消失中，狀態：Terminating
[user@minikube ~]$ kubectl get pods
NAME                              READY   STATUS        RESTARTS   AGE
hello-kitty                       1/1     Running       0          6d
hello-kitty-d7tfq                 1/1     Terminating   0          15m
hello-kitty-lk7lg                 1/1     Running       0          10h
hello-kitty-x94s2                 1/1     Running       0          10h
hello-kitty-zgrb8                 1/1     Running       0          15m
hello-minikube-7c77b68cff-ncqmn   1/1     Running       1          11d
hello-moto-6456dff9f-67jh5        1/1     Running       0          6d

# 觀察 pod 變化，確認副本 pod hello-kitty-XXXXX 剩下3個。
[user@minikube ~]$ kubectl get pods
NAME                              READY   STATUS    RESTARTS   AGE
hello-kitty                       1/1     Running   0          6d
hello-kitty-lk7lg                 1/1     Running   0          10h
hello-kitty-x94s2                 1/1     Running   0          10h
hello-kitty-zgrb8                 1/1     Running   0          16m
hello-minikube-7c77b68cff-ncqmn   1/1     Running   1          11d
hello-moto-6456dff9f-67jh5        1/1     Running   0          6d
```

### Scale Pod 方法差異

上述用了兩種方式進行 pod scale，`kubectl scale --replicas`跟`kubectl apply -f`  
_**scale pod**_ 目的與效果是相同的，但其實意義上，是有所不同的～  
`kubectl scale --replicas`指令，適合自己工作上運用，但自己也要清楚當前 replicas valus。  
`kubectl apply -f`是透過宣告式 yaml 檔案進行 replicas valus 管理，非常適合納入版控系統並搭配自動化腳本進行管理。進而更加適合團隊性質的協同合作需求。 

你會希望我做過什麼變動，我也必須讓大家知道做過哪些變動，**這是 DevOps 精神**。

{% hint style="info" %}
參考書籍：  
Kubernetes 建置與執行 書中 P.98~102 內容
{% endhint %}

