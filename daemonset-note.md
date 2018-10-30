# 淺談 DaemonSet，及相對 ReplicaSet 的差異

話說 ReplicaSet 是實現服務 AutoScale or Loadbalance 的方式，  
但是如需在 K8s node 叢集環境下確保特定 pod 在每個 node 上皆能夠實現相同服務，  
**DaemonSet** 反而是最佳選擇。   
典型的應用服務包括：

* 叢集儲存，比如 glusterd、ceph
* 日誌收集，比如 fluentd、logstash
* 系统監控，比如 Prometheus Node Exporter、collectd、New Relic agent、Ganglia gmond
* 系统程序，比如 kube-proxy、kube-dns

### DaemonSet 特性

* DaemonSet 會確保每個 node 上運行同一個 pod 服務～ 除非透過 nodeSelector。
* 例如系統常駐性監控服務，是非常適合透過 DaemonSet 實現副本需求。
* DaemonSet 如同 ReplicaSet 皆透過 Reconciliation Loops 可以確保目前 Status 是否符合預期 Spec。
* 當 K8s 新增 node 時，DaemonSet 會於 new node 上新增 pod 副本。

### 非 DaemonSet 特性

如果需要為了服務大量客戶流量，需要建立大量 pod 副本，  
並且無需考量與 node 耦合性問題， 透過 ReplicaSet 進行 service scale 是正確的選擇。

{% hint style="info" %}
參考書籍\文章出處：  
Kubernetes 建置與執行 書中 P.105~114 內容  
[https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)
{% endhint %}

