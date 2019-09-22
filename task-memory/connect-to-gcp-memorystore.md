# 如何連線至 GCP Memorystore

上一篇提到在 GCP**`Memorystore`**的服務中建立 Redis 執行個體，建立後當然需要使用它，此篇來說明如何讓應用程式使用 Redis 服務，後面再說明如何從開發者、維運人員之本機上連線至 Redis 服務。

### Redis 網路連線屬性

如下圖所示，GCP Memorystore Redis 服務僅提供內部網路 IP 網段。  


![](../.gitbook/assets/ying-mu-kuai-zhao-20190920-shang-wu-1.42.41.png)

此 Redis 服務可以從 GKE 上的應用程式 pod 發起連線。_\(此部分會獨立於另一篇說明\)_  
也可從 GCP Instance 上對此 Redis 進行連線，下面說明**開發者如何連線**，就與 GCP Instance 有所關聯。

### 開發者如何連線

如果要從開發者的 MacOS 本機上連線至雲端 Redis 服務，需要進行下面兩個步驟。

```text
# 1. 首先建立 GCP instances，此步驟只需進行一次即可，除非事後刪除了。
#    此步驟建立 GCP instances，名稱自訂例如redis-forwarder，關於--machine-type 可自選。
gcloud compute instances create redis-forwarder --machine-type=f1-micro

# 2. 透過上述 instance 進行 Redis 服務連線
gcloud compute ssh redis-forwarder -- -N -L 6380:10.0.0.3:6379
```

經過上述兩步驟後，即可從本機上`6380` listen port 進行 Redis 服務連線。

