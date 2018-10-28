# Pod 副本管理～ 描述 ReplicaSet 控制器

K8s 系統背後設計與運行著 Reconciliation Loops 觀念。  
**ReplicaSet** 就是一個實例功能。  
K8s Pod 定義中有此欄位 **Spec** 指的就是 desired state；  
那至於系統觀察到的 **Status** 又是意指 observed state。

### Reconciliation Loops

Reconciliation Loops 會一直運行著，一直觀察著 Pod 是否如預期般的運作著必要數量副本。   
假設 http pod 指定 replicas = 3，則因其它因素導致觀察到的 http pod 僅有 2，  
則，Reconciliation Loops \( ReplicaSet \)此機制就會讓 http pod 恢復為 3。

{% hint style="info" %}
Reconciliation Loops 中文譯：調節迴圈
{% endhint %}

### ReplicaSet

在現有運行中的多個 Pod 清單中，要針對哪一個 pod 進行副本管理，則需要透過 Label 進行篩選動作。 ReplicaSet 會透過 K8s API 取得 Pod 清單，並透過 Label 進行篩選，  
ReplicaSet 根據 K8s API 回傳的 Pod Status，進行相對應的 replica 動作。

* ReplicaSet 就像其他 K8s 物件一樣，透過宣告式進行定義。
* ReplicaSet 在`metadata.name`欄位中必須有唯一的名稱。
* `spec.replicas`欄位定義運行指定的 pod 之數量。
* `spec.containers`定義著 pod 所運行的 container 模板。
* ReplicaSet 定義範例：

```yaml
apiVersion: extensions/v1beta1
kind: ReplicaSet
metadata:
  name: hello-minikube
spec:
  replicas: 2
  template:
    metadata:
      labels:
        run: hello-minikube
  spec:
    containers:
      - name: hello-minikube
        image: "k8s.gcr.io/echoserver:1.10"
```

### 歷史洪流資訊

過去有個 ReplicationController 功能，與相較起來，除了其本質是相同的功能與應用，  
另外名稱不同外，現在主流 ReplicaSet 功能還多了 set-based selector。

現在主流 K8s 會建議透過 Deployments 控制器進行 Pod 副本管理，  
其本質就是由 Deployments 去調用 replicaSet 完成副本相關工作，  
但是多了重要功能：**rolling-update**  
我們在[這篇](https://fufu.gitbook.io/kk8s/first-pod-deployment#deployment)，有介紹過 Deployments 用途～

