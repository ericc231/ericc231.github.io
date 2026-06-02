---
title: "在K8S中安裝開發用IBM MQ"
date: 2022-06-29 22:20:13 +0800
categories: ["生活點滴", "軟體開發"]
tags: ["IBM MQ"]
---

首先建立一個namespace來安裝

```
kubectl create namespace ibmmq
```

建立一個PV提供MQ做使用

```
#pv.yaml內容
apiVersion: v1
kind: PersistentVolume
metadata:
  name: ibmmq-pv1
  labels:
    type: local
spec:
  storageClassName: ibmmq-storage-class
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    #在本機上開個資料夾給PV用
    path: "/someWhere"
```

```
kubectl apply -f pv.yaml
```

我們希望每次POD啟動都可以抓到相同的PV，所以使用StatefulSet，為了創建StatefulSet，需要先創建一個Headless Service。

```
#headless.yaml內容
apiVersion: v1
kind: Service
metadata:
  name: ibmmq-headless-server
  namespace: ibmmq
  labels:
    app: ibmmq
spec:
  ports:
  - name: ibmmq-port
    port: 1414
  - name: console-port
    port: 9443
  clusterIP: None
  selector:
    app: ibmmq
```

```
kubectl apply -f headless.yaml
```

再來是創建StatefulSet，Docker Hub上面的ibmcom/mq已經deprecated，現在移到IBM Container Registry。

```
#ibmmq.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: ibmmq
  namespace: ibmmq
spec:
  selector:
    # 必須與 ".spec.template.metadata.labels" 相同
    matchLabels:
      app: ibmmq
  serviceName: "ibmmq"
  replicas: 1
  template:
    metadata:
      # 必須與 ".spec.selector.matchLabels" 相同
      labels:
        app: ibmmq
    spec:
      securityContext:
        fsGroup: 1001
      terminationGracePeriodSeconds: 10
      containers:
      - name: ibmmq
        image: icr.io/ibm-messaging/mq:9.2.5.0-r1
        ports:
        - containerPort: 1414
          name: ibmmq
        - containerPort: 9443
          name: console
        # 指定將 pvc 掛載到特定的目錄上
        volumeMounts:
        - name: data
          mountPath: /mnt/mqm
        env:
        - name: LICENSE
          value: accept
        - name: MQ_QMGR_NAME
          value: QM_1
#指定admin跟app的密碼，測試用，這邊改用secret會比較合理
        - name: MQ_ADMIN_PASSWORD
          value: passw0rd
        - name: MQ_APP_PASSWORD
          value: passw0rd
  # 使用 persistent volume 來確保資料不會因為 pod reschedule 而消失
  # 以下是使用 volumeClaimTemplates + StorageClass 來完成
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "ibmmq-storage-class"
      resources:
        requests:
          storage: 5Gi
```

IBM MQ container會用UID 1001來執行([參考這](https://github.com/ibm-messaging/mq-container/blob/master/docs/security.md))，我們mount了/mnt/mqm來給IBM MQ使用，如果沒有改權限，就會無法正常運作，首先試了initContainer，總是會報錯誤，後來改用mount options，但mount options不支援local的模式，後來終於找到可以用securityContext解決，設定fsGroup: 1001，則/mnt/mqm的owner就會是1001。

再來我是用NodePort的模式試圖把1414跟9443開通給本機連線，不過minikube似乎無法直接這麼做，我還是只能使用port forward或minikube service來連接，所以下面這個service做了也沒效果。

```
apiVersion: v1
kind: Service
metadata:
  name: ibmmq-service
  namespace: ibmmq
  labels:
    app: ibmmq
spec:
  type: NodePort
  ports:
  - name: ibmmq-port
    protocol: "TCP"
    port: 1414
    nodePort: 30000
    targetPort: 1414
  - name: console-port
    protocol: "TCP"
    port: 9443
    nodePort: 30001
    targetPort: 9443
  selector:
    app: ibmmq
```

透過Lens Desktop來開Port Forward也很方便

![](https://ericchen231.wordpress.com/wp-content/uploads/2022/06/e688aae59c96-2022-06-29-e4b88be58d889.41.47.png?w=1024)

可以選擇直接打開browser跟使用https，比minikube service方便多了，不能指定https，打開browser就變成做白工。

![](https://ericchen231.wordpress.com/wp-content/uploads/2022/06/e688aae59c96-2022-06-29-e4b88be58d889.43.41.png?w=1024)

9443是內建web console的入口，Lens打開browser就會到console的登入畫面，預設帳號是admin，密碼是pod環境變數MQ\_ADMIN\_PASSWORD的內容，若沒有設定則預設passw0rd，成功登入後就會看到下面的畫面：

![](https://ericchen231.wordpress.com/wp-content/uploads/2022/06/e688aae59c96-2022-06-29-e4b88be58d889.48.44.png?w=1024)

再來要開1414的Port-Forwarding，測試IBM MQ Explorer是否可以正常連接，這個image啟動後會建立兩條channel

![](https://ericchen231.wordpress.com/wp-content/uploads/2022/06/e688aae59c96-2022-06-29-e4b88be58d8810.00.59-1.png?w=1024)
![](https://ericchen231.wordpress.com/wp-content/uploads/2022/06/e688aae59c96-2022-06-29-e4b88be58d8810.00.09.png?w=855)

成功了

![](https://ericchen231.wordpress.com/wp-content/uploads/2022/06/e688aae59c96-2022-06-29-e4b88be58d8810.06.20.png?w=1024)

用MQ Explorer試著放一筆訊息，然後web console來檢視

![](https://ericchen231.wordpress.com/wp-content/uploads/2022/06/e688aae59c96-2022-06-29-e4b88be58d8810.08.56.png?w=1024)

這次用到的範例YAML檔案可以在[這裡](https://github.com/ericc231/k8s-ibmmq)找到。
