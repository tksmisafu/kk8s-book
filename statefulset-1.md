# 發現、初談 StatefulSet

### StatefulSet 特色：

過去談過 Deployment、ReplicaSet 所創建的 Pod 皆屬於 stateless 的情況。  
但如要建立 stateful 特性的服務資源物件，可以透過 StatefulSet 完成建立。  
對於想要實現下列服務目的者，使用 StatefulSet 是最恰當的～

* 穩定的唯一性網路標誌       Stable, unique network identifiers.
* 穩定的持久化儲存服務       Stable, persistent storage.
* 有序性的實現部署和擴充   Ordered, graceful deployment and scaling.
* 有序性的滾動更新               Ordered, automated rolling updates.

以上，除了服務 stateful & stateless 重要差異之外，  
特別凸顯了 Stable 穩定、Ordered 有序的字眼，這就是與 Deployment 之間最大差別。

### StatefulSet 條件限制：

1. K8s 版本必須是 1.9 版本之後
2. 用於 Pod 的儲存必須由 PersistentVolume Provisioner 根据 storage class 進行配置或者管理員事先配置。
3. 刪除或者擴容 StatefulSet 時不會刪除相關的儲存區；這是為了確保資料的安全。
4. StatefulSets 目前需要 Headless Service 來負責 Pod 網路定義。
5. StatefulSets 刪除時，並不保證 pod 完美的終止服務。
6. StatefulSets 中實現 pod 的有序性和正常終止，可以在刪除之前將 StatefulSet 縮小到 0。

### 展示 StatefulSet 配置

配置中分三項說明：

1. Headless Service：名稱 nginx，用於控制網路
2. StatefulSet：名稱 web，有指定三個副本 replicas
3. volumeClaimTemplates：透過 PersistentVolumes provisioned 提供穩定的儲存服務

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx # has to match .spec.template.metadata.labels
  serviceName: "nginx"
  replicas: 3 # by default is 1
  template:
    metadata:
      labels:
        app: nginx # has to match .spec.selector.matchLabels
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "my-storage-class"
      resources:
        requests:
          storage: 1Gi
```

#### 注意 Pod Selector：

* 在 StatefulSet yaml 定義中，必須有 .spec.template.metadata.labels；
* 在 K8s 1.8 版本之前，Pod Selector 是可以被省略的；
* 在 K8s 1.9 版本之後，如沒有設定正確的 Pod Selector 是會造成建立 StatefulSet 失敗。

{% hint style="info" %}
參考書籍\文章出處：  
Kubernetes 建置與執行 書中 P.180~181 內容。  
[https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)  
[https://jimmysong.io/kubernetes-handbook/concepts/statefulset.html](https://jimmysong.io/kubernetes-handbook/concepts/statefulset.html)  
[https://feisky.gitbooks.io/kubernetes/concepts/statefulset.html](https://feisky.gitbooks.io/kubernetes/concepts/statefulset.html)
{% endhint %}

