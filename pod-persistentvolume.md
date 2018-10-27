# Pod 資料，如何持久化存放、讀取

當 Pod 重啟時或者刪除後，容器內的檔案都不會保留 \(重啟後不會保留先前產生的檔案\)，這就是 Container 的特性！  
但很多場景下，也是需要 “保留存檔”、“掛載現有資料” 等等需求～  
K8s 提供幾種方案，完成您的需求。

#### 這應用稱呼為：PersistentVolume

#### K8s 提供了以下幾種常見實現 PersistentVolume 方案：

1. emptyDir
2. hostDir
3. nfs
4. iscsi
5. cephfs
6. Rados Block Device
7. GCE Persistent Disk
8. AWS EBS Volume
9. Azure Data Disk

   族繁不及備載.....

如要完成持久化資料應用，在 Pod manifest 定義設定檔中，有兩個地方需定義：

* spec.volume：這定義 Pod 裡頭“所有容器”能夠存取“宿主主機”的磁碟區
* containers.volumeMounts：此定義是針對“個別容器”掛載磁區的目錄。

{% hint style="info" %}
參考書籍\文章出處：   
**Kubernetes 建置與執行** 書中 P.59~62 內容  
[https://kubernetes.io/docs/concepts/storage/volumes/](https://kubernetes.io/docs/concepts/storage/volumes/)
{% endhint %}



