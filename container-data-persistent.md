# Container Data Persistent

Container App 造就了應用服務可以更趨近微服務化、開發、測試與線上一致性等優點  
還帶來 service scale、rolling update 等靈活應用，但也因為靈活，所以面對資料持久化、資料落地更不可能不面對。

資料持久性、資料落地指的是，常態保存在特定的資料磁區中，可讀可寫並且不受 Container 生命週期影響，可確保資料的保存性、持久性。

繼此篇，今日介紹 **PersistentVolume \(PV\)** 與 **PersistentVolumeClaim \(PVC\)** 兩個物件。  
**PV** 物件是定義儲存區。  
**PVC** 物件是定義與使用 **PV** 儲存服務而賦予 Pod 使用。

**PV** 物件就形同是 K8s 的 node，決定 Volume 供應商並提供了相對資源給予上層 PVC 使用。   
**PVC** 物件就是相當於 K8s node 上的 Pod，Pod 消費 Node 資源，意即 Pod 請求 node CPU\RAM 資源，  
而 **PVC** 請求 **PV** 的資源大小和訪問模式。

會這樣的兩層 Volume 磁區物件，是為了保持 Pod 與 Volume 非耦合性關係的服務狀態。

### PersistentVolume

```yaml
# PersistentVolume 物件範例：
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0003
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  nfs:
    path: /tmp
    server: 172.17.0.2
```

#### PV `accessModes` 訪問模式：

* `ReadWriteOnce`（RWO）：最基本的方式，可讀可寫，但只支持被單個節點掛載。
* `ReadOnlyMany`（ROX）：提供只讀不可寫，可被多個節點掛載。
* `ReadWriteMany`（RWX）：提供可讀可寫，可被多個節點掛載。  不是每種存儲型態都支持這三種方式， 像共享方式，目前支持的還比較少，比較常用的是 NFS。 在 PVC 綁定 PV 時通常根據兩個條件來綁定，一個是存儲的大小，另一個就是訪問模式。

#### PV `persistentVolumeReclaimPolicy` 回收策略：

* `Retain`：不會刪除數據，保留 Volume（需要手動清理）
* `Recycle`：刪除數據，即 rm -rf /thevolume/\*（只有 NFS 和 HostPath 支持）
* `Delete`：刪除數據資源，比如刪除 AWS EBS 卷（只有 AWS EBS, GCE PD, Azure Disk 和 Cinder 支持）

### PersistentVolumeClaims

```yaml
# PersistentVolumeClaims 物件範例：
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 8Gi
  storageClassName: slow
  selector:
    matchLabels:
      release: "stable"
    matchExpressions:
      - {key: environment, operator: In, values: [dev]}
```

* `resources`：決定 Pod 掛載的磁區空間大小。

{% hint style="info" %}
參考文章出處：   
[https://kubernetes.io/docs/concepts/storage/persistent-volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes)  
[https://feisky.gitbooks.io/kubernetes/concepts/persistent-volume.html](https://feisky.gitbooks.io/kubernetes/concepts/persistent-volume.html)
{% endhint %}

