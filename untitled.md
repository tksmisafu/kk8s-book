---
description: notes
---

# kubectl describe nodes

```bash
afu@afu-P5440UA:~$ kubectl describe nodes
Name:               minikube
Roles:              master
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/hostname=minikube
                    node-role.kubernetes.io/master=
Annotations:        node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Wed, 17 Oct 2018 19:04:06 +0800
Taints:             <none>
Unschedulable:      false
Conditions:
  Type             Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----             ------  -----------------                 ------------------                ------                       -------
  OutOfDisk        False   Fri, 19 Oct 2018 00:39:31 +0800   Wed, 17 Oct 2018 19:03:59 +0800   KubeletHasSufficientDisk     kubelet has sufficient disk space available
  MemoryPressure   False   Fri, 19 Oct 2018 00:39:31 +0800   Wed, 17 Oct 2018 19:03:59 +0800   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure     False   Fri, 19 Oct 2018 00:39:31 +0800   Wed, 17 Oct 2018 19:03:59 +0800   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure      False   Fri, 19 Oct 2018 00:39:31 +0800   Wed, 17 Oct 2018 19:03:59 +0800   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready            True    Fri, 19 Oct 2018 00:39:31 +0800   Wed, 17 Oct 2018 19:03:59 +0800   KubeletReady                 kubelet is posting ready status
Addresses:
  InternalIP:  10.0.2.15
  Hostname:    minikube
Capacity:
 cpu:                2
 ephemeral-storage:  16888216Ki
 hugepages-2Mi:      0
 memory:             2038624Ki
 pods:               110
Allocatable:
 cpu:                2
 ephemeral-storage:  15564179840
 hugepages-2Mi:      0
 memory:             1936224Ki
 pods:               110
System Info:
 Machine ID:                 1aad573e92344dec83c3eba307d94f42
 System UUID:                8F37BF6F-4CC9-4B86-B2D6-96EA6311280B
 Boot ID:                    4f5fdf19-5517-4ab0-a7c3-70a8add0baeb
 Kernel Version:             4.15.0
 OS Image:                   Buildroot 2018.05
 Operating System:           linux
 Architecture:               amd64
 Container Runtime Version:  docker://17.12.1-ce
 Kubelet Version:            v1.10.0
 Kube-Proxy Version:         v1.10.0
Non-terminated Pods:         (10 in total)
  Namespace                  Name                                     CPU Requests  CPU Limits  Memory Requests  Memory Limits
  ---------                  ----                                     ------------  ----------  ---------------  -------------
  kube-system                coredns-c4cffd6dc-lhmm5                  100m (5%)     0 (0%)      70Mi (3%)        170Mi (8%)
  kube-system                etcd-minikube                            0 (0%)        0 (0%)      0 (0%)           0 (0%)
  kube-system                kube-addon-manager-minikube              5m (0%)       0 (0%)      50Mi (2%)        0 (0%)
  kube-system                kube-apiserver-minikube                  250m (12%)    0 (0%)      0 (0%)           0 (0%)
  kube-system                kube-controller-manager-minikube         200m (10%)    0 (0%)      0 (0%)           0 (0%)
  kube-system                kube-dns-86f4d74b45-s2bvw                260m (13%)    0 (0%)      110Mi (5%)       170Mi (8%)
  kube-system                kube-proxy-lkvjx                         0 (0%)        0 (0%)      0 (0%)           0 (0%)
  kube-system                kube-scheduler-minikube                  100m (5%)     0 (0%)      0 (0%)           0 (0%)
  kube-system                kubernetes-dashboard-6f4cfc5d87-j4d42    0 (0%)        0 (0%)      0 (0%)           0 (0%)
  kube-system                storage-provisioner                      0 (0%)        0 (0%)      0 (0%)           0 (0%)
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource  Requests     Limits
  --------  --------     ------
  cpu       915m (45%)   0 (0%)
  memory    230Mi (12%)  340Mi (17%)
Events:
  Type    Reason                   Age                    From                  Message
  ----    ------                   ----                   ----                  -------
  Normal  Starting                 5m50s                  kubelet, minikube     Starting kubelet.
  Normal  NodeHasSufficientDisk    5m50s (x6 over 5m50s)  kubelet, minikube     Node minikube status is now: NodeHasSufficientDisk
  Normal  NodeHasSufficientMemory  5m50s (x6 over 5m50s)  kubelet, minikube     Node minikube status is now: NodeHasSufficientMemory
  Normal  NodeHasNoDiskPressure    5m50s (x6 over 5m50s)  kubelet, minikube     Node minikube status is now: NodeHasNoDiskPressure
  Normal  NodeHasSufficientPID     5m50s (x5 over 5m50s)  kubelet, minikube     Node minikube status is now: NodeHasSufficientPID
  Normal  NodeAllocatableEnforced  5m50s                  kubelet, minikube     Updated Node Allocatable limit across pods
  Normal  Starting                 4m39s                  kube-proxy, minikube  Starting kube-proxy.
afu@afu-P5440UA:~$
```

