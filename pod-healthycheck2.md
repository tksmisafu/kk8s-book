# Pod 的健康檢查方式 Part-2

### **Readiness 功能**

上一篇提到 Liveness pod check 功能，今日此篇介紹另一個 pod check 功能：**Readiness**

這兩者 check 功能是有差異的～  
Liveness 是在 pod 已經運行狀態下並且開放請求流量時，進行 pod check 任務。  
Readiness 是反過來，就在 Pod 未接受請求流量前就開始進行 pod check，並確認 check ok 之後，pod 才加入 service 行列中開始接受請求流量。  
此功能目的就是：**確認 pod 已經準備就緒！**

其配置方式與 Liveness 是相同地：

```text
# 以下僅是配置示意範本
spec:
  containers:
  - image: k8s.gcr.io/echoserver:1.10
    readinessProbe:
      httpGet:
        path: /healthy
        port: 8080
      initialDelaySeconds: 1
      periodSeconds: 10
      timeoutSeconds: 1
      failureThreshold: 3
```

### 配置說明

上篇沒提到的配置說明，今日完整補充  
Probe 配置中有很多依場景需求而定，盡可能滿足 liveness 和 readiness 的檢查功能：

* `initialDelaySeconds`：容器啟動後幾秒之後進行第一次探測任務。
* `periodSeconds`：執行探測的頻率間隔，默認是10秒，最小1秒。
* `timeoutSeconds`：探測超時時間，默認1秒，最小1秒。
* `successThreshold`：繼上次失敗後，最少連續探測成功次數才被認定為成功，默認是 1；
  * 對於 **liveness** 必須是 1，最小值是 1。 
* `failureThreshold`：連續探測失敗多少次才被認定為失敗，默認是 3，最小值是 1；
  * **liveness** 達標情況下將重啟 pod。
  * **readiness** 在此標準下，認定 pod 未準備就緒。

#### HTTP 應用中進行 httpGet 檢查，有其他配置選項：

* `host`：連線的主機名，預設連線到 pod IP。
* `scheme`：連線使用的 schema，默認HTTP。
* `path`: 請求 HTTP server 的 path。
* `httpHeaders`：定義 HTTP request header。
* `port`：容器內所啟用的 HTTP TCP Port，介於 1 和 65525 之間。

有關 HTTP probe，由 kubelet 針對所指定的`path、port`發送 HTTP 請求進行檢查。  
預設，Kubelet 將 probe 發送到容器的 IP 地址，除非設置了`host`欄位而覆蓋了預設值。  
在大多數情況下，是不用設置`host`欄位。

有一種情況下您可以設置它～  
假設容器在 _127.0.0.1 Listen service_，並且 Pod`hostNetwork`欄位是`true`，此時，`httpGet host`是該設置為`127.0.0.1`。  
如果 pod 依賴於虛擬主機，這可能是更常見的情況，不建議使用`host`，而應該利用`httpHeaders`去定義 _host header_。

{% hint style="info" %}
參考書籍\文章出處：   
**Kubernetes 建置與執行** 書中 P.55  P.81 內容  
[https://k8smeetup.github.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/](https://k8smeetup.github.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/) [https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/\#define-readiness-probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/#define-readiness-probes)
{% endhint %}

