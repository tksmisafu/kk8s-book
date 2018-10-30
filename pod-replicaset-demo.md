# Pod 副本管理～ 實作篇

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
[user@minikube ~]$ kubectl apply -f rs-hello-kitty.yaml
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

進行擴展 Pod

