# 再談 StatefulSet

上一篇提到 StatefulSet 是與 Deployment 不同之處，今日要針對下面兩個特點，特別獨立出此篇來敘述。

* 穩定的唯一性網路標誌 **Stable, unique network identifiers.**
* 穩定的持久化儲存服務 **Stable, persistent storage.**

StatefulSet Pods 具有一組唯一識別的 ordinal，穩定的網路狀態與儲存服務，  
無需在意 pod 是在哪些 K8s node 上運行。

### ordinal index

* 對於有 N replicas 的 StatefulSet，此識別序號必然是唯一性，有其重要性。
* 每個 StatefulSet Pod 皆賦予一組整數序號，從 0 開始至 N-1。

### Stable Network ID

* 每個 StatefulSet Pod 的 name 作為前綴加上 ordinal，建構出 hostname = `$(statefulset name)-$(ordinal)`
* 上一篇 StatefulSet 範例中，將創建出三個名稱是 web 的 Pod，其 hostname = web-0、web-1、web-2

### Domain

* StatefulSet 可以透過 Headless Service 來決定旗下的 Pod Domain。
* 此 Domain 的格式：`$(service name).$(namespace).svc.cluster.local` ; 其中 “`cluster.local`” 是 Cluster Domain。
* 每個 Pod 建立時，將取得一個 DNS subdomain 格式：
  * `$(podname).$(governing service domain)` ;
  * 其中 governing service 取之於 StatefulSet yaml 定義中的 serviceName 欄位。

{% hint style="warning" %}
官網解說的，上面敘述的方法使我自己也好複雜，需要心靜的腦筋消化～  
下面有簡單的解說方式～
{% endhint %}

#### StatefulSet 中每個 Pod DNS Domain 格式如下：

`$(statefulset name)-$(ordinal).$(service name).$(namespace).svc.cluster.local`

```text
$(statefulset name) 是 StatefulSet 的名字。
-$(ordinal) 是 StatefulSet Pod 賦予的一組整數序號，從 0 開始至 N-1。
$(service name) 是 Headless Service 的名字。
$(namespace) 是服務所在的 namespace，Headless Service 和 StatefulSet 需在相同的 namespace。
.cluster.local 是 Cluster Domain。
```

### Stable Storage

* 在K8s **StatefulSet** 中，透過 VolumeClaimTemplate 所定義的規格建立出 PV。
* 當相關的 Pod 重新被調度至其他 K8s node 上，則會掛載與 **PersistentVolume Claim** 關聯的 PV。
* 需要注意：
  * 依此 **PersistentVolume Claim** 關聯的 PV 在 Pods or StatefulSet 刪除時，不會刪除 PV 內資料，需要手動移除。

### 補充說明：什麼是 StorageClass

* **StorageClass** 透過 PV 的類別\(等級\)名稱，來定義 PV 的類別屬性，方便管理者去分門別類儲存屬性，例如 High/Slow IO
* 建立 StorageClass 時，管理員設置名稱與相關參數，一但建立後不可再對其進行更新。
* 一個有指定`StorageClassName`的 PV，只許被相同`StorageClassName`的 PVC 綁定請求。
* 一個沒指定`StorageClassName`的 PV，只有無指定`StorageClassName`的 PVC 綁定請求。

K8s API server 上的 **DefaultStorageClass** 是否啟用，攸關到 StorageClass default 的變因。

* 如啟用，可以設定 default **StorageClassName**，當 PVC 中沒有設置`StorageClassName`，則會套用 default **StorageClassName**。
* 如啟用，但沒設定 default`StorageClassName`，當 PVC 創建時 K8s 會回應。
* ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
* 如沒啟用，則就沒有 StorageClass 概念；
* 在此情況，没有`storageClassName`的 PVC 與 「`storageClassName: ""`的 PVC 」 的處理方式相同。

{% hint style="info" %}
參考書籍\文章出處：   
Kubernetes 建置與執行 書中 P.180~181 內容。 [https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) [https://jimmysong.io/kubernetes-handbook/concepts/statefulset.html](https://jimmysong.io/kubernetes-handbook/concepts/statefulset.html) [https://feisky.gitbooks.io/kubernetes/concepts/statefulset.html](https://feisky.gitbooks.io/kubernetes/concepts/statefulset.html)​
{% endhint %}



