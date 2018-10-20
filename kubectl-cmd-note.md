---
description: >-
  kubectl 是 K8s client 命令工具指令，他可用來建立物件、服務、更新；此命令工具指令是與 K8s apiserver
  互動達成你所需的工作目標。以下，來介紹個別基本指令。
---

# 常見的 kubectl 指令

### Kubectl 與 物件觀念

1. K8s 所有資源都是 RESTful API 格式組成的，每個資源皆稱呼為物件 object。
2. 每個物件都有唯一的 http 路徑，例如： [https://you.k8s.com/api/v1/namespace/default/pods/my-pod](https://you.k8s.com/api/v1/namespace/default/pods/my-pod)
3. 一個 pod 名稱是：my-pod 。
4. 此 pod 物件位於 namespace:default 空間中。

### 檢視 K8s API 物件

* 要取得特定資源，kubectl 指令方式是 `kubectl get <資源名稱> <物件名稱>` 
* 預設下，`kubectl get` 僅輸出簡單明瞭之單行資訊
* 如要檢視稍多資訊，可透過 `-o wide` 旗標，在該行中取得稍多資訊。
* 如要檢視完整資訊，可透過 `kubectl get pods my-pod -o json | -o yaml` 取得。
* 如要檢視完整資訊，另一方式指令 `kubectl describe <資源名稱> <物件名稱>` 。
* 如要移除首行資訊，可搭配使用 `--no-headers` 旗標。
* 如要只提取特定欄位，可搭配使用 `-o jsonpath --template={.status.podIP}` 。

### 新增、更新、移除 K8s 物件

* K8s API 物件都可用 JSON or YAML 格式檔案進行定義。
* 檔案可以是 apiserver 回應的查詢結果，也可傳送至 apiserver 執行一次請求。
* 建立物件指令：`kubectl apply -f http.yaml`
* 如果 http.yaml 物件重新修改過，可以重複執行上述指令。 
* _**也可線上編輯物件，但非常不建議這麼做，這牽涉到團隊協同合作情境。**_ 
* 線上編輯指令：`kubectl edit <資源名稱> <物件名稱>`
* 刪除物件指令：`kubectl delete -f http.yaml`
* 也可以是：`kubectl delete <資源名稱> <物件名稱>`
* 刪除指令只要一送出，該物件即刻刪除，K8s 系統並不會額外提醒刪除動作。

### 標注與註解物件

* K8s 物件是可以透過 annotate、label 兩指令進行標注與註解。 
* 新增標注方式：`kubectl label pods my-pod color=green`
* 移除標注方式：`kubectl label pods my-pod -color`
* 觀察標注？

### 除錯指令

{% hint style="info" %}
**Log 不得少，不然怎麼除錯呢？** 
{% endhint %}

* 查看 log 方式：`kubectl log my-pod`
* 如果一個 pod 中有多個 container，可以使用`-c`另行指定 container。
* 上述指令執行，如同 Linux`tail`指令即刻跳出輸出，如要讓 log 不斷輸出，可以加上`-f`命令。
* 想進入 container bash 環境進行除錯， 可以執行指令：`kubectl exec -it my-pod --bash`
* 想複製 pod container 中的檔案，可以執行指令 `kubectl cp <pod-name>:/path/abc.txt ~/abc.txt`
* 反過來也可複製到 container 中：`kubectl cp ~/xyz.txt <pod-name>:/path/xyz.txt`

{% hint style="info" %}
參考書籍：  
**Kubernetes 建置與執行** 書中 P.38~42 內容
{% endhint %}

