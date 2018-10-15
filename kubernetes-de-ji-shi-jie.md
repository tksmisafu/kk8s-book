---
description: 什麼是 Kubernetes？先認識下吧～
---

# Kubernetes 的基礎世界

Kubernetes 的用途，用來管理基於 Container 基礎服務，並具有高可用性、叢集環境、自動擴容與可促進應用程式 zero downtime 的調度系統。

舉例說，程式設計師基於功能需求，依賴 Framework 去開發出應用系統，則～ 系統架構師基於 Container 服務，藉由 Kubernetes 架構出適合的系統運作環境。

Kubernetes 起始於Google 公司Borg專案中，並且於Google公司雲平台中運作多年，進而演進出開源的kubernetes。

Kubernetes 系統有數項角色、名詞是不可能忽略的：

* controller-manager
* Scheduler
* Etcd

  **上述三項是 Master node 具有的 Container**

* Kubernetes proxy
* Kubernetes DNS
* Kubernetes UI
* **上述三項是 K8s集群元件**
* NameSpace
* Context
* API服務 \ Scheduler \ Etcd
* POD
* Label \ Annotation
* Service
* ReplicaSet \ DaemonSet
* ConfigMap \ Secret
* Deployment

{% hint style="info" %}
參考文章\出處：

* [https://en.wikipedia.org/wiki/Kubernetes\#Architecture](https://en.wikipedia.org/wiki/Kubernetes#Architecture)
* [https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/](https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/)
* [https://ai.google/research/pubs/pub43438](https://ai.google/research/pubs/pub43438)
{% endhint %}



