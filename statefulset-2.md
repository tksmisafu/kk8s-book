# 再談 StatefulSet

上一篇提到 StatefulSet 是與 Deployment 不同之處，今日特別獨立出此篇來敘述。   
StatefulSet Pods 具有一組唯一識別的 ordinal，穩定的網路狀態與儲存服務，無需在意 pod 是在哪些 K8s node 上運行。

### ordinal index

* 對於有 N replicas 的 StatefulSet，此識別序號必然是唯一性，有其重要性。
* 每個 StatefulSet Pod 皆賦予一組整數序號，從 0 開始至 N-1。

### Stable Network ID

* 每個 StatefulSet Pod 的 name 作為前綴加上 ordinal，建構出 hostname = $\(statefulset name\)-$\(ordinal\)
* 上一篇 StatefulSet 範例中，將創建出三個名稱是 web 的 Pod，其 hostname = web-0、web-1、web-2

### Domain

* StatefulSet 可以透過 Headless Service 來決定旗下的 Pod Domain。
* 此 Domain 的格式：$\(service name\).$\(namespace\).svc.cluster.local ; 其中 “cluster.local” 是 Cluster Domain。
* 每個 Pod 建立時，將取得一個 DNS subdomain 格式：
  * $\(podname\).$\(governing service domain\) ;
  * 其中 governing service 取之於 StatefulSet yaml 定義中的 serviceName 欄位。

{% hint style="warning" %}
官網解說的，上面敘述的方法使我自己也好複雜，需要心靜的腦筋消化～  
下面有簡單的解說方式～
{% endhint %}

#### StatefulSet 中每個 Pod DNS Domain 格式如下：

$\(statefulset name\)-$\(ordinal\).$\(service name\).$\(namespace\).svc.cluster.local

```text
$(statefulset name) 是 StatefulSet 的名字。
-$(ordinal) 是 StatefulSet Pod 賦予的一組整數序號，從 0 開始至 N-1。
$(service name) 是 Headless Service 的名字。
$(namespace) 是服務所在的 namespace，Headless Service 和 StatefulSet 需在相同的 namespace。
.cluster.local 是 Cluster Domain。
```

{% hint style="info" %}
參考書籍\文章出處：   
Kubernetes 建置與執行 書中 P.180~181 內容。 [https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) [https://jimmysong.io/kubernetes-handbook/concepts/statefulset.html](https://jimmysong.io/kubernetes-handbook/concepts/statefulset.html) [https://feisky.gitbooks.io/kubernetes/concepts/statefulset.html](https://feisky.gitbooks.io/kubernetes/concepts/statefulset.html)​
{% endhint %}



