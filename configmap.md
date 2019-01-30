# 認識 configMap

## Examples

```bash
  # Create a new configmap named my-config based on folder bar
  kubectl create configmap my-config --from-file=path/to/bar

  # Create a new configmap named my-config with specified keys instead of file basenames on disk
  kubectl create configmap my-config --from-file=key1=/path/to/bar/file1.txt --from-file=key2=/path/to/bar/file2.txt

  # Create a new configmap named my-config with key1=config1 and key2=config2
  kubectl create configmap my-config --from-literal=key1=config1 --from-literal=key2=config2

  # Create a new configmap named my-config from the key=value pairs in the file
  kubectl create configmap my-config --from-file=path/to/bar

  # Create a new configmap named my-config from an env file
  kubectl create configmap my-config --from-env-file=path/to/bar.env
  
  # Delete a configmap named my-config
  kubectl delete configmap my-config
```

### 建立 config

```bash
kubectl create configmap my-config --from-file=my-config.txt \
    --from-literal=another-param=config1 \
    --from-literal=extra-param=config2

```

### 建立 pod

```yaml
# 11-133-kuard-config.yaml
apiVersion: v1
kind: Pod
metadata:
  name: kuard-config
spec:
  containers:
    - name: test-container
      image: gcr.io/kuar-demo/kuard-amd64:1
      imagePullPolicy: Always
      command:
        - "/kuard"
        - "$(EXTRA_PARAM)"
      env:
        - name: ANOTHER_PARAM
          valueFrom:
            configMapKeyRef:
              name: my-config
              key: another-param
        - name: EXTRA_PARAM
          valueFrom:
            configMapKeyRef:
              name: my-config
              key: extra-param
      volumeMounts:
        - name: config-volume
          mountPath: /config
  volumes:
    - name: config-volume
      configMap:
        name: my-config
  restartPolicy: Never
```

### 生效、連線 pod

```text
# 生效 pod
kubectl apply -f 11-133-kuard-config.yaml

# 連線 pod
kubectl port-forward --address 0.0.0.0 kuard-config 30333:8080
```

### 查看 ConfigMap

```yaml
# kubectl get cm my-config -o yaml
apiVersion: v1
data:
  another-param: config1
  extra-param: config2
  my-config.txt: |
    another-param = value1
    extra-param = value2
kind: ConfigMap
metadata:
  creationTimestamp: "2019-01-29T03:51:25Z"
  name: my-config
  namespace: default
  resourceVersion: "1426607"
  selfLink: /api/v1/namespaces/default/configmaps/my-config
  uid: 2644b22a-2379-11e9-8813-08002730aeb3
```

## 回頭解析 Pod YAML

### 環境變數

引用環境變數，定義在`env - valueFrom`範圍中，這會參照該`my-config (ConfigMap)`裡頭的 key 作為環境變數。

```yaml
      env:
        - name: ANOTHER_PARAM
          valueFrom:
            configMapKeyRef:
              name: my-config
              key: another-param
        - name: EXTRA_PARAM
          valueFrom:
            configMapKeyRef:
              name: my-config
              key: extra-param
```

### Command - 引用變數

使用命令列參數，可透過上述`valueFrom`環境變數引用，  
K8s 將使用`$(Environment Variable)`語法作為變數表達與引用。

```yaml
  containers:
    - name: test-container
      image: gcr.io/kuar-demo/kuard-amd64:1
      imagePullPolicy: Always
      command:
        - "/kuard"
        - "$(EXTRA_PARAM)"
```

檔案系統

Pod 內透過`volumeMounts`定義了一個磁碟區命名為_**config-volume**_，並掛載於 _**/config**_ 路徑。  
實際磁碟來源，是依據`ConfigMap`內的 _**my-config**_ 來建立的檔案系統。  
在操作中進入 _**/config**_ 路徑內，會看見`ConfigMap`的每個項目建立了檔案或目錄，

```yaml
      volumeMounts:
        - name: config-volume
          mountPath: /config
  volumes:
    - name: config-volume
      configMap:
        name: my-config
```

