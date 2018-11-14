# 驗證自建的 K8s

今日雖然是鐵人賽的最後一篇，但不是 KK8s 的最後一篇～  
學不完，只怕學不來 XDD

#### 安心一下，先看下目前叢集狀態

```bash
[vagrant@kk8s-1 ~]$ kubectl get nodes -o wide
NAME     STATUS   ROLES    AGE   VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION              CONTAINER-RUNTIME
kk8s-1   Ready    master   47h   v1.12.2   192.168.42.191   <none>        CentOS Linux 7 (Core)   3.10.0-862.2.3.el7.x86_64   cri-o://1.11.8
kk8s-2   Ready    <none>   24h   v1.12.2   192.168.42.192   <none>        CentOS Linux 7 (Core)   3.10.0-862.2.3.el7.x86_64   cri-o://1.11.8
kk8s-3   Ready    <none>   24h   v1.12.2   192.168.42.193   <none>        CentOS Linux 7 (Core)   3.10.0-862.2.3.el7.x86_64   cri-o://1.11.8

```

```bash
[vagrant@kk8s-1 ~]$ kubectl get all --all-namespaces
NAMESPACE     NAME                                 READY   STATUS    RESTARTS   AGE
kube-system   pod/coredns-576cbf47c7-dgktb         1/1     Running   0          47h
kube-system   pod/coredns-576cbf47c7-vv7rc         1/1     Running   0          47h
kube-system   pod/etcd-kk8s-1                      1/1     Running   1          2d
kube-system   pod/kube-apiserver-kk8s-1            1/1     Running   2          2d
kube-system   pod/kube-controller-manager-kk8s-1   1/1     Running   0          2d
kube-system   pod/kube-flannel-ds-amd64-cnsl2      1/1     Running   0          47h
kube-system   pod/kube-flannel-ds-amd64-qdqls      1/1     Running   0          25h
kube-system   pod/kube-flannel-ds-amd64-sr4zj      1/1     Running   0          25h
kube-system   pod/kube-proxy-bjvkj                 1/1     Running   0          25h
kube-system   pod/kube-proxy-hrsdw                 1/1     Running   0          2d
kube-system   pod/kube-proxy-kl959                 1/1     Running   0          25h
kube-system   pod/kube-scheduler-kk8s-1            1/1     Running   2          2d

NAMESPACE     NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)         AGE
default       service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP         2d
kube-system   service/kube-dns     ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP   2d

NAMESPACE     NAME                                     DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR                     AGE
kube-system   daemonset.apps/kube-flannel-ds-amd64     3         3         3       3            3           beta.kubernetes.io/arch=amd64     47h
kube-system   daemonset.apps/kube-flannel-ds-arm       0         0         0       0            0           beta.kubernetes.io/arch=arm       47h
kube-system   daemonset.apps/kube-flannel-ds-arm64     0         0         0       0            0           beta.kubernetes.io/arch=arm64     47h
kube-system   daemonset.apps/kube-flannel-ds-ppc64le   0         0         0       0            0           beta.kubernetes.io/arch=ppc64le   47h
kube-system   daemonset.apps/kube-flannel-ds-s390x     0         0         0       0            0           beta.kubernetes.io/arch=s390x     47h
kube-system   daemonset.apps/kube-proxy                3         3         3       3            3           <none>                            2d

NAMESPACE     NAME                      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kube-system   deployment.apps/coredns   2         2         2            2           2d

NAMESPACE     NAME                                 DESIRED   CURRENT   READY   AGE
kube-system   replicaset.apps/coredns-576cbf47c7   2         2         2       2d
```

```text
查看 API 資訊：https://192.168.42.191:6443/api/v1
```

#### 今日來做個基本驗證，採用過去用過的 app yaml

```text
kubectl apply -f https://raw.githubusercontent.com/kubernetes/website/master/content/en/examples/application/nginx-app.yaml
```

#### 觀察部署狀況

```bash
[vagrant@kk8s-1 ~]$ kubectl get all -n default
NAME                            READY   STATUS    RESTARTS   AGE
pod/my-nginx-5c689d88bb-68kbs   1/1     Running   0          35s
pod/my-nginx-5c689d88bb-wnmr9   1/1     Running   0          35s
pod/my-nginx-5c689d88bb-x7qqg   1/1     Running   0          35s

NAME                   TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
service/kubernetes     ClusterIP      10.96.0.1     <none>        443/TCP        46h
service/my-nginx-svc   LoadBalancer   10.99.29.64   <pending>     80:32031/TCP   35s

NAME                       DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/my-nginx   3         3         3            3           35s

NAME                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/my-nginx-5c689d88bb   3         3         3       35s

#######
[vagrant@kk8s-1 ~]$ kubectl get endpoints -n default
NAME           ENDPOINTS                                   AGE
kubernetes     192.168.42.191:6443                         46h
my-nginx-svc   10.244.1.2:80,10.244.1.3:80,10.244.2.2:80   3m12s

```

```bash
# 上面狀態中，可以看見 service/my-nginx-svc NodePort = 32031
# 查看下自己的系統網路層 Listen 資訊
[vagrant@kk8s-1 ~]$ ss -anut |grep LISTEN |grep 32031
tcp    LISTEN     0      128      :::32031                :::*
[vagrant@kk8s-1 ~]$
[vagrant@kk8s-1 ~]$
[vagrant@kk8s-1 ~]$ sudo lsof -i :32031
COMMAND     PID USER   FD   TYPE  DEVICE SIZE/OFF NODE NAME
kube-prox 13473 root    6u  IPv6 9834316      0t0  TCP *:32031 (LISTEN)
[vagrant@kk8s-1 ~]$

```

來訪問 [http://master-ip:32031](http://master-ip:32031)  
失敗中...

```text
失敗中...
失敗中...
失敗中...
afu@Ichirode-MacBook-Pro$ curl http://192.168.42.191:32031
curl: (55) getpeername() failed with errno 22: Invalid argument
```

到這個節骨眼，過程中的除錯仍徒勞無功，看來很需要再回顧自己不足之處～ 自己還得要加油！

今日在除錯的過程中，發現未來有幾個面向，還要持續探索～

* K8s Dashboard
* cAdvisor
* Heapster**、**InfluxDB、grafana
* CNI
* CRI

謝謝朋友這次鼓吹我參加鐵人賽，不然壓根子沒想到參加鐵人賽活動！  
我這次參加的鐵人賽的概念，就是將讀書、讀文章的心得寫下來，所以很多都是概念文～  
至於文章的順序，雖是隨想而決定，但基本上會是將當下有關聯的 key 去探索研讀，相對不像是標準網誌般有章節順序概念去呈現篇文。

謝謝這三十天的機會，後續就鞭策自己！

