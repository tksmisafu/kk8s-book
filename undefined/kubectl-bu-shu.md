# kubectl 部署筆記

```bash
# 部署
[user@minikube ~]$ kubectl run hello-minikube --image=k8s.gcr.io/echoserver:1.10 --port=8080
kubectl run --generator=deployment/apps.v1beta1 is DEPRECATED and will be removed in a future version. Use kubectl create instead.
deployment.apps/hello-minikube created
######################################
[user@minikube ~]$ kubectl get pod
NAME                              READY   STATUS    RESTARTS   AGE
hello-minikube-7c77b68cff-ncqmn   1/1     Running   0          1m
######################################
[user@minikube ~]$ kubectl get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   1d
######################################
[user@minikube ~]$ kubectl get deployments
NAME             DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
hello-minikube   1         1         1            1           1m
######################################
[user@minikube ~]$ kubectl get daemonSets
No resources found.

####################################################################################

# Expose it as a new Kubernetes Service
[user@minikube ~]$ kubectl expose deployment hello-minikube --type=NodePort
service/hello-minikube exposed
######################################
[user@minikube ~]$ kubectl get pod
NAME                              READY   STATUS    RESTARTS   AGE
hello-minikube-7c77b68cff-ncqmn   1/1     Running   0          4m
######################################
[user@minikube ~]$ kubectl get services
NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
hello-minikube   NodePort    10.97.219.147   <none>        8080:31472/TCP   32s
kubernetes       ClusterIP   10.96.0.1       <none>        443/TCP          1d
######################################
[user@minikube ~]$ kubectl get deployments
NAME             DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
hello-minikube   1         1         1            1           5m
######################################
[user@minikube ~]$ kubectl get daemonSets
No resources found.
[user@minikube ~]$
```

