---
description: 什麼是Docker，什麼是Container ?  這確實需要比K8s更早認識一下～
---

# What is Docker ? What is Container ?

#### 所以我先說說 Container

```text
在使用上，Docker 大家或許比起 Container 還熟悉 
我因應趨勢潮流，或許也算跟風“以正視聽”，嘗試表達以下等號、不等號
*Docker == Container、Container != Docker*

Container 就是基於 Linux CGroups、NameSpace 等內核區隔技術並且共用內核情況下，將應用程式創造於獨立沙盒執行環境。
Container 也不是 VM Hypervisor。
Container 技術目標，是建立一個可以共用內核，相容Linux標準安裝程序又可獨立運作應用程式的環境。
Container 授權是 GNU LGPLv2.1+ 。
Container ，就是 Linux Container，簡稱 LXC。
```

#### 參考文章\出處：

* [https://zh.wikipedia.org/wiki/LXC](https://zh.wikipedia.org/wiki/LXC)
* [https://linuxcontainers.org/lxc/introduction/](https://linuxcontainers.org/lxc/introduction/)

#### 回來說說 Docker

Docker 的訴求，官網寫得很清楚～ [https://www.docker.com/](https://www.docker.com/)

> #### Build, Manage and Secure Your Apps Anywhere.   Your Way.     The Dev to Ops Choice for Container Platforms.

Docker現今已經是一個平台，意即Container 是容器化技術，運行容器化服務的平台Docker是其一，K8S是其二。  
所以看官們，能懂此篇文章一開始的等號規則嗎？

```text
Docker 來自於容器技術開發公司 dotCloud。
約略2013年起，Docker開源專案與眾多世界性開發者貢獻，合力致力於發展 Docker技術。
2015年6月之後Docker 公司將 images、runtime 代碼開源給OCI基金會，幫助 Container發展標準化。
2017年4月，Docker 公司成立了新開源專案Moby，將過去的公司心血（也是眾多世界性開發者心血）
          明確劃分為企業版(DockerEE)、社群版(DockerCE)、開源專案(Moby)
原本的專案名稱：Docker 立馬變成 Moby～
詳細資訊，請詳閱 iTHome 報導(連結如下)
```

#### 參考文章\出處：

* [https://www.docker.com/resources/what-container](https://www.docker.com/resources/what-container)
* [https://www.ithome.com.tw/news/113899](https://www.ithome.com.tw/news/113899)

