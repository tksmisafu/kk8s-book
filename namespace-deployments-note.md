# NameSpace、Deployments 概念說明

### NameSpace \(命名空間\) 概念

K8s 利用命名空間將特定服務、物件自畫一個空間，可使與其他服務、物件分屬在不同活動空間中，達到互不影響服務的隔離作用。  
先來看看 K8s 預設有哪些 namespace\(命名空間\)

```bash
[user@minikube ~]$ kubectl get namespace
NAME          STATUS   AGE
default       Active   1d
kube-public   Active   1d
kube-system   Active   1d
```

```text
default：這是預設命名空間，創建物件、服務時，如未指定空間則就分配於此空間中。
kube-system：此命名空間是 K8s 系統自帶物件之運作區。
kube-public：此命名空間其特性是[公用]，每個用戶都可存取此空間下的資源服務。
```

K8s 可透過 context 變更預設的命名空間，可以透過指令`kubectl config current-context`得知目前 context。  
變更 context 方式如下：

```bash
[user@minikube ~]$ kubectl config set-context ABC --namespace=abc
Context "ABC" created.

[user@minikube ~]$ kubectl config use-context ABC
Switched to context "ABC".

[user@minikube ~]$ kubectl config get-contexts
CURRENT   NAME       CLUSTER    AUTHINFO   NAMESPACE
          ABC                              abc
*         minikube   minikube   minikube
########
# 上述動作會寫入在本機 ~/.kube/config 設定擋中。 
########
```

{% hint style="info" %}
說明參考頁/書籍：  
[https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)  
**Kubernetes 建置與執行** 書中 P.37~38內容
{% endhint %}

### Deployments 概念

1. Deployments 是個提供陳述性\(宣告式\)方式去驅動 Pod  ReplicaSet 之物件。
2. 只需要明確陳述你的服務目標狀態，deployments 就會幫 pod  replicaset 現況改變成目標狀態。
3. 用於管理服務的發佈版本、RollingUpdate、Rollback、Scale up\down、healthcheck。
4. 因此，實現服務部署、更新不會出現停機或錯誤變得很簡單。

{% hint style="info" %}
說明參考夜頁/書籍：  
[https://kubernetes.io/docs/concepts/workloads/controllers/deployment/](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)  
**Kubernetes 建置與執行** 書中 P.149~152內容
{% endhint %}



