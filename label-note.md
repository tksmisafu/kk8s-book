# 來說說 Label

來説説過往篇章裡頭，沒有特別說明的功能：**Label**  
我對此功能的心得算是：標籤  
ㄜ...... **Label** 中文譯就是 標籤

**Label** 此功能是幫助大家針對所管理的各類物件：**Pod**、**ReplicaSet**、**Deployment** 等等，賦予 **Label**  
如此，就可以針對部署的服務，賦予暱稱管理功能，比如：

* 針對環境：test、stage、canary、prod
* 針對版本：version-1、version-2
* 針對地區：Asia、Europe、Africa、America、Oceania
* 針對產品：ERP、eCommerce、Cloud

這些標籤，也可以複數使用，具體舉例～  
eCommerce 電子商務系統中的任何服務，皆有個 Label = eCommerce  
其中負責網頁服務的 http pod 物件，有地區性及環境等等區別，那電子商務系統的 http pod，就會有下列的清單：

```text
Pod-name | Labels
-----------------------------------------------
eC-http  | env=tests , PROD=eC
eC-http  | env=stage , PROD=eC
eC-http  | env=prod  , PROD=eC , loca=Asia
eC-http  | env=prod  , PROD=eC , loca=Europe
eC-http  | env=prod  , PROD=eC , loca=America

```

當您進行 K8s 管理時，或者自動化流程裡頭，皆可透過指定 **Label** 去篩選物件，而後進行管理、流程目的。以上，用自己的想法嘗試了說明 Label 用途。

**Label** 本質是 Key/value 的組合，賦予物件標籤，達到任意地附加識別資訊。  
可以組織性的標記、交叉索引，以表示應用服務具有意義性的群組概念。

Label 此 Key/Value 的組合語法，分為兩個部分說明：

```text
Key 表達方式可以是 “名稱”，例如 env、loca。
    “名稱” 的第一與最後一個字元，必須是英數，字元之間可以是(-)(_)(.)。
    “名稱” 不可超過63個字元。
    
    另一個表達方式：“前綴”+“名稱”，例如 kubernetes.io/cluster
    “前綴”部分一定是 domain.dns 格式，“前綴”+“名稱”中間必須用斜線(/)區隔著。
    “名稱”，是必要的項目，“前綴”是選擇性項目。

Value 僅需是值或者字元呈現，並與“名稱”有相同規範。

```

{% hint style="info" %}
參考書籍\文章出處：   
Kubernetes 建置與執行 書中 P.65~66 內容
{% endhint %}

