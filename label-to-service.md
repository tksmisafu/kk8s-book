# 從 Label 再回頭談 Service

K8s 裡頭最小服務單位就是 Pod，Pod 的生命週期會因眾多因素而有變化，  
可能消失了，可能自動建立 pod，也可能 restart pod～  
那...Service 需要管理到 Pod 是否運作中嗎？

Ans 是不需要的，在過去幾篇提到建立 Pod 的方式中，都有提到定義 **Label**，  
例如 `app=nginx` 在 **Service** 定義中，透過 Label Selector 來影響了服務流量要傳送給哪些 pod 上。

### Service 定義範例 1：

```yaml
kind: Service
apiVersion: v1
metadata:
  name: web-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8011
```

上述 Service`web-service` 中，會在叢集中 Listen 80 埠端口，並且將流量傳送至後端符合 `app=nginx` 條件的 **pod** \(Listen 8011\)。

### Service 定義範例 2：

```yaml
kind: Service
apiVersion: v1
metadata:
  name: hello-minikube
spec:
  selector:
    app: hello-minikube
    env: demo
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080
```

上述 Service`hello-minikube` 中，會在叢集中 Listen 8080 埠端口，並且將流量傳送至後端同時符合`app=hello-minikube`&`env=demo`兩條件的 **pod** \(Listen 8080\)。  
以上，描述了 **Service** 與 **Label** 之間重要關係～

那有個新問題：Service 如何透過 Label 去得知哪些符合條件 **pod** 的真實網路、埠端口資訊呢？  
（如果不用知道 **pod** 真實資訊，流量網路層就無法傳遞囉！）

K8s 提供了 **Endpoint** API 方式，每當創建 **Service** 時，就會依據 **Label** 條件去建立 **Endpoint** 資訊紀錄，並且此 **Endpoint** 資訊紀錄會一直 watch **pod** 狀態，當 **pod** 移除、新建等情況時，皆會即時更新 **Endpoint** 資訊紀錄。

* 所以 **Endpoint** 資訊紀錄，有三大要素：Label、Pod\_IP、Pod\_Port
* 記得，**Endpoint** 是因 service 中運用了 **Label selector** 而產生出的資訊紀錄。
* 反過來，**Service** 如沒運用 **Label selector** 條件，則是不會建立 **Endpoint**。

### Endpoint 資訊紀錄

```yaml
[user@minikube ~]$ kubectl describe endpoints hello-minikube
Name:         hello-minikube
Namespace:    default
Labels:       run=hello-minikube
Annotations:  <none>
Subsets:
  Addresses:          172.17.0.4,172.17.0.6
  NotReadyAddresses:  <none>
  Ports:
    Name     Port  Protocol
    ----     ----  --------
    <unset>  8080  TCP
```

{% hint style="info" %}
參考書籍\文章出處：   
Kubernetes 建置與執行 書中 P.78-79、P86-87 內容。  
[https://jimmysong.io/kubernetes-handbook/concepts/service.html](https://jimmysong.io/kubernetes-handbook/concepts/service.html)  
[https://tachingchen.com/tw/blog/kubernetes-service/](https://tachingchen.com/tw/blog/kubernetes-service/)
{% endhint %}

