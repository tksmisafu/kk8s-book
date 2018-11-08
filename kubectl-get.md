# kubectl get 筆記

```bash
afu@afu-P5440UA:~$ kubectl get daemonSets --namespace=kube-system
NAME         DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
kube-proxy   1         1         1       1            1           <none>          1d
afu@afu-P5440UA:~$
```

```bash
afu@afu-P5440UA:~$ kubectl get deployments --namespace=kube-system
NAME                   DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
coredns                1         1         1            1           1d
kube-dns               1         1         1            1           1d
kubernetes-dashboard   1         1         1            1           1d
```

```bash
afu@afu-P5440UA:~$ kubectl get services --namespace=kube-system
NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)         AGE
kube-dns               ClusterIP   10.96.0.10      <none>        53/UDP,53/TCP   1d
kubernetes-dashboard   ClusterIP   10.109.75.255   <none>        80/TCP          1d
```

```bash
kubectl get pod --namespace=kube-system
NAME                                    READY   STATUS    RESTARTS   AGE
coredns-c4cffd6dc-lhmm5                 1/1     Running   1          1d
etcd-minikube                           1/1     Running   0          16m
kube-addon-manager-minikube             1/1     Running   1          1d
kube-apiserver-minikube                 1/1     Running   0          16m
kube-controller-manager-minikube        1/1     Running   0          16m
kube-dns-86f4d74b45-s2bvw               3/3     Running   4          1d
kube-proxy-lkvjx                        1/1     Running   0          15m
kube-scheduler-minikube                 1/1     Running   1          1d
kubernetes-dashboard-6f4cfc5d87-j4d42   1/1     Running   3          1d
storage-provisioner                     1/1     Running   3          1d
```

```bash
afu@afu-P5440UA:~$ kubectl get pod
No resources found.

afu@afu-P5440UA:~$ kubectl get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   1d

afu@afu-P5440UA:~$ kubectl get deployments
No resources found.

afu@afu-P5440UA:~$ kubectl get daemonSets
No resources found.
```

