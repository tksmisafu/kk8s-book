# 如何連線至 GCP Memorystore

上一篇提到在 GCP**`Memorystore`**的服務中建立 Redis 執行個體，建立後當然需要使用它，此篇來說明如何讓應用程式使用 Redis 服務，後面再說明如何從開發者、維運人員之本機上連線至 Redis 服務。

### Redis 網路連線屬性

如下圖所示，GCP Memorystore Redis 服務僅提供內部網路 IP 網段。  
此 Redis 服務可以從 GKE 上的應用程式 pod 發起連線。

![](../.gitbook/assets/ying-mu-kuai-zhao-20190920-shang-wu-1.42.41.png)

### 開發者如何連線

如果要從開發者的 MacOS 本機上連線至雲端 Redis 服務，需要進行下面兩個步驟。

```text
# 1. 首先建立 GCP instances，此步驟只需進行一次即可
gcloud compute instances create port-forwarder --machine-type=f1-micro

# 2. 透過上述 instance 進行 Redis 服務連線
gcloud compute ssh port-forwarder -- -N -L 6380:10.0.0.3:6379
```



