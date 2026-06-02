---
title: "在K8S中搭建Redis Cluster"
date: 2022-06-30 21:43:33 +0800
categories: ["K8S"]
---

![](https://ericchen231.wordpress.com/wp-content/uploads/2022/05/redis_cluster_k8s-2.png?w=215)

## Redis Cluster

你不必等到具備卓越能力再開始行動，但你必須展開行動才能臻至卓越，這次試著在K8S中搭建Redis Cluster，並理解每個步驟所代表的意義。。

在進入到Redis叢集(Cluster)之前，先來看看Redis除了單體Standalone跟叢集以外，還有哪幾種執行模式

## 主從模式(Master-Slave)

在主從模式下，寫入的動作只能對Master，Slave只能進行讀取的動作，一個Master可以有很多個Slave，Master會把資料同步給Slave，Master如果故障了，資料就沒辦法寫入，這模式單純是用讀寫分離來加速讀取的效率，如果需要HA或Failover，這模式並不合適．Slave再多都不會增加整體的記憶體使用空間．

## 哨兵模式(Sentinel)

哨兵模式顧名思義就是透過站哨的衛兵監控著Master，如果Master無法運作了，就會讓某個Slave提升為Master，但只靠一個哨兵的觀察就判定狀態會顯得太過武斷，而雙數的哨兵投票會很沒效率，可能會出現票數相同的情形，此時就要讓某個哨兵可以投兩票，然後再投一次，所以哨兵數量最好是單數，投票一次就會有結果．也因此，哨兵最少數量是3個．

![](https://ericchen231.wordpress.com/wp-content/uploads/2022/05/redis_sentinel.png?w=400)

哨兵模式示意圖

哨兵模式簡單說就是增加了HA能力的主從模式，所以相同的，多Slave並不會增加整體記憶體空間．

## 叢集模式(Cluster)

叢集模式解決了單體(Standalone)、主從(Master-Slave)、哨兵(Sentinel)等模式的記憶體空間無法擴充的問題．

> 搭建Redis Cluster至少要有三組Master-Slave的組合
>
> 也就是要有3個Master Node跟3個Slave Node

最基本的叢集模式由三主三從組成，主節點故障後，從節點會補上，但若補上的從節點又故障了，也就是整個叢集的主節點少於3個，此時整個叢集將無法運作．

![](https://ericchen231.wordpress.com/wp-content/uploads/2022/05/redis_cluster_basic.png?w=656)

最基本的叢集配置

為了解決上述的單點失敗問題，Redis叢集有個Slave節點飄移機制，以下圖為例，當Master 1故障時，Slave 1接手變成Master節點，此時叢集就有了單點失敗的風險，叢集會檢查哪裡有多的Slave節點，如果有多的，會移動一個Slave過去，也就是Slave 3、Slave 4、Slave 5其中一個會被移去當Slave 1的Slave．

![](https://ericchen231.wordpress.com/wp-content/uploads/2022/05/redis_cluster_span.png?w=666)

接著試著在K8S (Minikube)中搭建Redis Cluster

## 先建立一個namespace來安裝redis

```
kubectl create namespace redis-cluster
```

![](https://ericchen231.wordpress.com/wp-content/uploads/2022/05/create_namespace.png?w=670)

因為POD重新啟動就會變回Image原來的樣子，我們必須有個能保存AOF或RDB的儲存區，有6個redis node，所以我們先建立6個PV (Persistent Volume)，存取模式是ReadWriteOnce，確保每個PV只會被分配到一個PVC，hostPath的部分是Node上的資料夾，用Local File System只能在one node cluster中使用，依照實際狀況調整．產生一個pv.yaml。（因為我們這裡是用minikube，hostPath不存在也不會出錯，minikube會自己產生一個出來。

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: redis-cluster-pv1
  labels:
    type: local
spec:
  storageClassName: redis-cluster-storage-class
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/redis1"
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: redis-cluster-pv2
  labels:
    type: local
spec:
  storageClassName: redis-cluster-storage-class
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/redis2"
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: redis-cluster-pv3
  labels:
    type: local
spec:
  storageClassName: redis-cluster-storage-class
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/redis3"
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: redis-cluster-pv4
  labels:
    type: local
spec:
  storageClassName: redis-cluster-storage-class
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/redis4"
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: redis-cluster-pv5
  labels:
    type: local
spec:
  storageClassName: redis-cluster-storage-class
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/redis5"
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: redis-cluster-pv6
  labels:
    type: local
spec:
  storageClassName: redis-cluster-storage-class
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/redis6"
```

把PV建起來

```
kubectl apply -f pv.yaml
```

![](https://ericchen231.wordpress.com/wp-content/uploads/2022/05/e688aae59c96-2022-05-26-e4b88ae58d8812.02.59.png?w=940)

我們希望每個POD在reschedule時，都能mount相同的pvc，所以要做成StatefulSet，得先做個headless service。

```
#headless.yaml
apiVersion: v1
kind: Service
metadata:
  name: redis-headless-server
  namespace: redis-cluster
  labels:
    app: redis-cluster
spec:
  ports:
  - name: redis-port
    port: 6379
  clusterIP: None
  selector:
    app: redis-cluster
```

```
kubectl apply -f headless.yaml
```

用ConfigMap做redis的設定檔

```
#configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: redis-conf
  namespace: redis-cluster
data:
  redis.conf: |+
    appendonly yes
    cluster-enabled yes
    cluster-config-file /var/lib/redis/nodes.conf
    dir /var/lib/redis
    port 6379
```

```
kubectl apply -f configmap.yaml
```

接著要把StatefulSet建立起來了

```
#redis.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis
  namespace: redis-cluster
spec:
  selector:
    matchLabels:
      app: redis-cluster
  serviceName: "redis-cluster"
  replicas: 6
  template:
    metadata:
      labels:
        app: redis-cluster
    spec:
      containers:
      - name: redis
        image: "redis:6.2.7"
        command:
          - "redis-server"
        args:
          - "/etc/redis/redis.conf"
          - "--protected-mode"
          - "no"
          - "--cluster-announce-ip"
          - "$(MY_POD_IP)"
        ports:
          - name: redis
            containerPort: 6379
            protocol: "TCP"
          - name: cluster
            containerPort: 16379
            protocol: "TCP"
        volumeMounts:
          - name: "redis-conf"
            mountPath: "/etc/redis"
          - name: "redis-data"
            mountPath: "/var/lib/redis"
        env:
        - name: MY_POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
      volumes:
      - name: "redis-conf"
        configMap:
          name: "redis-conf"
          items:
            - key: "redis.conf"
              path: "redis.conf"
  volumeClaimTemplates:
  - metadata:
      name: redis-data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "redis-storage-class"
      resources:
        requests:
          storage: 5Gi
```

```
kubectl apply -f redis.yaml
```

輸入指令，可以看到POD有正常啟動

![](https://ericchen231.wordpress.com/wp-content/uploads/2022/06/e688aae59c96-2022-06-30-e4b88be58d884.58.20.png?w=1024)

試著去看log，可以發現因為還沒進行cluster的指令設定，所以cluster還沒運作起來

![](https://ericchen231.wordpress.com/wp-content/uploads/2022/06/e688aae59c96-2022-06-30-e4b88be58d884.57.54.png?w=1024)

先取得所有POD的ip

![](https://ericchen231.wordpress.com/wp-content/uploads/2022/06/e688aae59c96-2022-06-30-e4b88be58d885.13.57.png?w=1024)

執行創建redis cluster的指令

```
kubectl -n redis-cluster exec -it redis-0 -- redis-cli --cluster create  172.17.0.6:6379 172.17.0.7:6379 172.17.0.8:6379 172.17.0.9:6379 172.17.0.10:6379 172.17.0.11:6379 --cluster-replicas 1
```

![](https://ericchen231.wordpress.com/wp-content/uploads/2022/06/e688aae59c96-2022-06-30-e4b88be58d889.17.24.png?w=1024)

redis自己分配3個master，3個slave

輸入yes後，會自己建立完成，對cluster做個簡單的壓測

```
kubectl -n redis-cluster exec -it redis-0 -- redis-benchmark -n 1000000 -t set,get -P 16 -q -h 172.17.0.6 -p 6379 --cluster
```

![](https://ericchen231.wordpress.com/wp-content/uploads/2022/06/e688aae59c96-2022-06-30-e4b88be58d889.34.47.png?w=1024)

這次使用的檔案可以在[GitHub](https://github.com/ericc231/k8s-redis-cluster)找到。
