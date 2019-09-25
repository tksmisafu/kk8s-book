# HELM 工具用途

HELM 是用來管理 Kubernetes 應用服務工具，透過定義Charts、安裝、更新複雜的應用服務。  
其用途類似 **Linux** 世界中的`APT`、`YUM`軟體管理套件。

## 特色

HELM 具備下列特性：

1. 透過`Chart`作為應用程式安裝包。
2. 定義眾多的 **Kubernetes** 資源物件於`Chart`中。
3. 透過`Repository`管理與分享眾多的`Chart`應用程式。
4. ​具備版本管理。
5. `Values.yaml` 可抽離出、管理應用程式的設定值。
6. 透過`helm`指令，簡易部署、更新應用程式在 **Kubernetes** 上。

## HELM 軟體庫\(Repository\)

Helm Repository list

* [https://hub.helm.sh/](https://hub.helm.sh/)
* [https://github.com/helm/charts](https://github.com/helm/charts)

## 概觀與元件

### Tiller Server

安裝於 **Kubernetes** 上，主要負責`helm`元件與`Kubernetes API`服務之間的溝通。

### helm client

指令`helm`用於建立、更新、管理`Chart`，藉由`Tiller server`元件將`chart`應用程式服務安裝於 **Kubernetes** 上。

### Repository

此為Chart的儲存倉庫，可版本管理、可分享、可自創。

### Release

透過`helm Chart`安裝於 **Kubernetes** 之中的應用程式稱為`release`，安裝後會自行產出`release name`，相同`chart`安裝兩遍以上，皆有個別不同的`release name`，當然也可自定義。



