# Service 續集之 Cluster IP、Kube-proxy、LoadBalancer

讀過了幾遍及翻了網路文章，才有這個開門見山的心得：  
cluster ip 是在建立 service 物件時由 K8s API 伺服器所分配。

### Cluster IP

**cluster ip** 是虛擬靜態IP，當被分配 IP 後就不再異動，除非刪除、重建 **service** 物件時，  
才會有所變動（重新分配）。  
**service cluster ip** 實現了負載平衡功能，賦予流量傳送到 Endpoint 內對象 pod。

如不需要或不需負載平衡，無需此單獨的 Service IP。  
可以經由指定 Cluster IP（`spec.clusterIP`）的值為 "`None`" 來創建 Headless Service。

```yaml
# Headless Service 範例
apiVersion: v1
kind: Service
apiVersion: v1
metadata:
  name: hello-minikube
spec:
  clusterIP: None
```

上述 **cluster ip** 是 **service** type 之一，支援的型態如下： 

* **ClusterIP**          &lt;&lt; 此篇主軸
* NodePort         &lt;&lt; 之前[這篇文章](https://fufu.gitbook.io/kk8s/first-service)有提到。 
* LoadBalancer 
* ExternalName

### Kube-proxy

實現 Service 這個抽象層功能（服務探索、負載平衡）的角色是 **kube-proxy**。

* **kube-proxy** 會在每一個 K8s node 上運作，會透過 K8s API 監視叢集中的 service 物件與 Endpoint。
* **kube-proxy** 會在 node 中藉由 **iptables 模式**撰寫規則，完成轉遞流量目的（負載平衡），service 越多則 iptables 規則也越多。
* **iptables 模式**也因為流量封包會透過核心層 iptables 轉遞，故效能的損耗問題不得不關注。
* **iptables 模式**是實現 Kube-proxy 方式之一，在 K8s v1.11 新版本之後新增支援 **ipvs 模式**。
* **ipvs 模式**是為了解决 iptables 模式的性能問題，採用增量式更新 iptables 規則，並且可保證 service 更新期間連線不中斷。（以上次研讀網路資訊得來，自己未實測過）

**Kube-Proxy iptables 模式**[解說圖](https://d33wubrfki0l68.cloudfront.net/27b2978647a8d7bdc2a96b213f0c0d3242ef9ce0/e8c9b/images/docs/services-iptables-overview.svg)

![](.gitbook/assets/k8s_kube-proxy-mode-iptables.svg)

### kube-proxy 的不足

* kube-proxy 目前只有支持 TCP 和 UDP，不支持 HTTP，並且也沒有健康檢查機制。
* 這些得需透過另一個方式 Ingress Controller 來解決問題。

{% hint style="warning" %}
這一篇的研讀，讓我更加明瞭 Service 是怎麼一回事情，也很感謝下列官方、網友文章的解說，讓我有所學習與突破。
{% endhint %}

### 關於 LoadBalancer

LoadBalancer 此 service type 適用於雲端 Kubernetes 環境中，例如 GKE，  
在 K8s service 部署於雲端環境中，如需暴露 service 供外界存取，即需用此。  
設定方式，僅須將 `type` 指定為 `LoadBalancer`。

另外，雖然需要暴露 service 供外部存取，但如僅需開放特定來源存取，可透過 `loadBalancerSourceRanges`參數指定來源 IP，即可限度性的開放服務。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  type: LoadBalancer
  loadBalancerSourceRanges:
  - 1.2.3.4/32
  - 5.6.7.8/32
```

{% hint style="info" %}
參考書籍\文章出處：   
Kubernetes 建置與執行 書中 P.88-89 內容。  
[https://kubernetes.io/docs/concepts/services-networking/service/](https://kubernetes.io/docs/concepts/services-networking/service/) [https://feisky.gitbooks.io/kubernetes/concepts/service.html](https://feisky.gitbooks.io/kubernetes/concepts/service.html) [https://feisky.gitbooks.io/kubernetes/components/kube-proxy.html](https://feisky.gitbooks.io/kubernetes/components/kube-proxy.html)  
關於 LoadBalancer、loadBalancerSourceRanges  
[https://kubernetes.io/docs/tasks/access-application-cluster/configure-cloud-provider-firewall/](https://kubernetes.io/docs/tasks/access-application-cluster/configure-cloud-provider-firewall/)
{% endhint %}

