---
description: Master node 有什麼狠角色，各別是什麼用途，透過此篇初步認識。
---

# K8s Master node Component 介紹

當透過 minikube 完成 Local K8s 環境建置後，下指令：`kubectl get componentstatuses` 可查看到 K8s 三個主要狠角色。

```bash
[user@minikube ~]$ kubectl get componentstatuses
NAME                 STATUS    MESSAGE              ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-0               Healthy   {"health": "true"}
```

### Master node component 介紹

#### Controller-manager

* 負責管理叢集內運作狀態的控制器。
* 其最主要運作目標：確保K8s運作良好，頭好壯壯健康程度100%。
* 管理範疇包含 node\repilication object\endpoints object\service account\API Token\NameSpace。

{% hint style="info" %}
**Kubernetes 建置與執行** 書中P.5內容，有敘述著 K8s 是具有自我修復性的系統，這就是 Controller-manager 最大的目的所在。
{% endhint %}

#### Scheduler

* 負責調度 pod 要在哪個 node 上運作，調度會依據**叢集資源**、**調度策略**，透過**調度演算**而決定 pod 運行在哪些 node 上。
* 具體的說～ scheduler 是 K8s 集群系統中運行的調度程序，負責收集、統計及分析叢集中所有節點的資源狀態，而後依此決定將新的 pod 選擇出在適當的 node 上運行服務。

#### etcd-0

* etcd-0：儲存叢集內所有的資料，即是所謂的資料儲存庫；是Key / value 資料型態。

#### _上述三項，都是運行在 Master node 上的三大角色，但別忽略了一件事情，起始一個新服務，是誰驅動\(調用\)這三個角色？_

### 是「Kube-apiserver」

* Master node 本身具有 Kube-apiserver 服務，其角色就是扮演 Master gateway～
* 對外，提供 kubectl client 給予資源操作入口。
* 對內，起於服務需求驅動 K8s node 完成服務建置。

說明參考頁：  
[https://kubernetes.io/docs/concepts/overview/components/\#master-components  
](https://kubernetes.io/docs/concepts/overview/components/
)**Kubernetes 建置與執行** 書中P.30內容

