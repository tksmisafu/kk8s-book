# Pod 的健康檢查方式

### **健康檢查進程**

K8s 運行應用程式的容器時，K8s 會利用**健康檢查進程**方式促進容器維持運作中～  
健康檢查進程只是確保 Container 內的應用程式進程能夠一直運行，如沒運行就進行重啟。  
但是，僅檢查程序 “是否運行” 是不足夠的，例如進程 deadlock 了，雖進程運行中但無法回應請求，這在**健康檢查程序**是無法判斷出問題的，且仍認為正常運行中...囧

### Liveness 探測器

K8s 提供了 Liveness 探測器功能，可以針對應用程式的邏輯進行請求，並期望獲得正常回應，藉此判斷應用程式是否合理的運作中。 

* 此 Liveness 探測器功能是定義於 Pod manifest 檔案中。
* 每個 Pod 的 Liveness 功能是分開個別定義的。
* livenessProbe 支援三種模式：
  * exec：利用命令腳本方式
  * httpGet: 透過 http 協定回應
  * tcpSocket：嘗試進行 TCP Socket 連線

### Liveness 定義方式

```yaml
# Manifest 檔案
spec:
  containers:
  - image: k8s.gcr.io/echoserver:1.10
    livenessProbe:
      httpGet:
        path: /healthy
        port: 8080
      initialDelaySeconds: 1
      periodSeconds: 10
      timeoutSeconds: 1
      failureThreshold: 3
```

### Liveness 應用場景

1. 可在容器內執行腳本或程式，定義`exec`執行腳本，探測後回傳 zero 退出狀態碼，則視為探測正常運作。
2. 針對 http 服務可以定義`httpGet`請求，藉此獲得 http response 狀態，判斷此 http 程序是否正常運作。
3. 針對 TCP Socket 連線狀態進行探測，可以定義`tcpSocket`。

  
   


