# Pod 的資源請求、上限

K8s Pod 裡頭有兩種不同用途的資源定義：

* **`request`**：此為針對 Container 可使用的“基本資源”定義。
* **`limit`**：此為針對 Container 可使用的“資源上限”定義。

### Request

* 在 Pod Manifest 中定義了此資源，K8s 平台即會針對此 Container 給予基本資源保證。
* 此定義是針對個別 Container 而定，Pod 的資源即是裡頭 Container 之總和。
* K8s 會針對 Pod 進行資源調度，所依據的資源可用即是此 request！
* 承上述，故而確保 Pod Container 運行時有足夠的資源可用。
* request 特色舉例：
  * K8s **node-1** 是2核心 2GB 記憶體，可以部署兩個 Pod 各一個 Container \(`cpu 800m / memory 512Mi`\)
  * 當相同的 Pod 要部署第三個，則因 K8s **node-1** 已經飽和 \(cpu 剩餘資源不足\)，K8s 調度器將第三個 Pod 往 **node-2** 進行部署。
  * 當 K8s **node-1** 資源未飽和情況下，裡頭兩個 Pod 即可分配到`cpu 1000m / memory 1024Mi`資源，當然只是定義中最低限度是`cpu 800m`。
* 因為 request 的保證性，因此對於高附載需求情況下，需要有足夠得資源，此功能顯得更加重要。

### Limit

當 Pod Manifest 中定義了`limit`時，K8s 底層核心確保 Container 運作上不超過定義中的上限。

* 當 K8s **node-1** 僅運行一個 Pod container \(`cpu 1500m / memory 1024Mi`\)，即使 **node-1** 仍有閒置的資源，仍不為此 container 可用的資源。
* 如果沒有定義 CPU、RAM Limit 則 Container 可用資源是無上限的，實際依據 node 資源為上限。

### 配置方式、單位說明

```yaml
spec:
  containers:
  - image: k8s.gcr.io/echoserver:1.10
    resources:
      requests:
        memory: "512Mi"
        cpu: "800m"
      limits:
        memory: "1024Mi"
        cpu: "1.5"
```

CPU 以 mili cpu 為單位表示，以 value + m 後綴表達，例如 1000mili cpu 以 1000m 表達～  
1000m = 1 cpu = 1 核心意思；500m = 0.5 核心～

RAM 以 byte 為計算單位，可以透過 G, M, K, Gi, Mi, Ki 作為後綴表達單位，例如：  
`128974848 = 129e6 = 129M = 123Mi`

{% hint style="info" %}
參考書籍/文章出處：  
**Kubernetes 建置與執行** 書中 P.56~59  
[https://k8smeetup.github.io/docs/tasks/configure-pod-container/assign-cpu-ram-container/](https://k8smeetup.github.io/docs/tasks/configure-pod-container/assign-cpu-ram-container/)
{% endhint %}



