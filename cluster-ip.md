# Service 續集之 Cluster IP

讀過了幾遍及翻了網路文章，才有這個開門見山的心得：  
cluster ip 是在建立 service 物件時由 K8s API 伺服器所分配。

cluster ip 是虛擬靜態IP，當被分配 IP 後就不再異動，除非刪除、重建 service 物件時，  
才會有所變動（重新分配）。  
service cluster ip 實現了負載平衡功能，賦予流量傳送到 Endpoint 內對象 pod。

如不需要或不需負載平衡，無需此單獨的 Service IP。  
可以經由指定 Cluster IP（spec.clusterIP）的值為 "None" 來創建 Headless Service。

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

上述 cluster ip 是 service type 之一，支援的型態如下： 

* ClusterIP          &lt;&lt; 此篇主軸
* NodePort         &lt;&lt; 之前這篇文章有提到。 
* LoadBalancer 
* ExternalName

