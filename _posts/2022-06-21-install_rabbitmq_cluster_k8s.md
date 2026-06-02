---
title: "在K8S中安裝RabbitMQ Cluster"
date: 2022-06-21 00:07:11 +0800
categories: ["生活點滴", "軟體開發"]
tags: ["RabbitMQ"]
---

RabbitMQ官方有釋出一個名為RabbitMQ Cluster Kubernetes Operator的工具來簡化在K8S中安裝RabbitMQ Cluster的作業，官方說明文件可以參考[這裡](https://www.rabbitmq.com/kubernetes/operator/operator-overview.html)。安裝指令：

```
kubectl apply -f https://github.com/rabbitmq/cluster-operator/releases/latest/download/cluster-operator.yml
```

![](https://ericchen231.wordpress.com/wp-content/uploads/2022/06/199bc8c6-9cf7-4595-92d6-ecad6700cfd1.jpeg?w=1024)

瞬間就安裝完成了，接著再測試hello-world的範例：

```
kubectl apply -f https://raw.githubusercontent.com/rabbitmq/cluster-operator/main/docs/examples/hello-world/rabbitmq.yaml
```

![](https://ericchen231.wordpress.com/wp-content/uploads/2022/06/4a948660-07cd-4e73-ae03-8b3659c0db22.jpeg?w=1024)

也是一下子就完成了，但這個範例只部署了一個pod，把replica設定成3個可能會比較有感。觀察一下log，確認是否有在運作：

```
kubectl logs hello-world-server-0
```

![](https://ericchen231.wordpress.com/wp-content/uploads/2022/06/e688aae59c96-2022-06-20-e4b88be58d8810.10.06.png?w=1024)
![](https://ericchen231.wordpress.com/wp-content/uploads/2022/06/e688aae59c96-2022-06-20-e4b88be58d8810.11.41.png?w=1024)

這個operator會把預設使用者帳密設定到secret中：

![](https://ericchen231.wordpress.com/wp-content/uploads/2022/06/e688aae59c96-2022-06-20-e4b88be58d8810.20.16.png?w=1024)

可以用以下指令解出來：

```
username="$(kubectl get secret hello-world-default-user -o jsonpath='{.data.username}' | base64 --decode)"
echo "username: $username"
password="$(kubectl get secret hello-world-default-user -o jsonpath='{.data.password}' | base64 --decode)"
echo "password: $password"
```

也可以在建立Cluster時指定帳密，可以參考GitHub[這範例](https://github.com/rabbitmq/cluster-operator/tree/main/docs/examples/set-login-password-username)。

接著測試是否可以正常進入RabbitMQ的web管理介面，For測試目的，直接用port forward的方式

```
kubectl port-forward "service/hello-world" 15672
```

打開瀏覽器，進入https://localhost:15762/

![](https://ericchen231.wordpress.com/wp-content/uploads/2022/06/e688aae59c96-2022-06-20-e4b88be58d8810.31.58.png?w=1024)

可以對剛建立的RabbitMQ做一下簡單的壓力測試：

```
username="$(kubectl get secret hello-world-default-user -o jsonpath='{.data.username}' | base64 --decode)"
password="$(kubectl get secret hello-world-default-user -o jsonpath='{.data.password}' | base64 --decode)"
service="$(kubectl get service hello-world -o jsonpath='{.spec.clusterIP}')"
kubectl run perf-test --image=pivotalrabbitmq/perf-test -- --uri amqp://$username:$password@$service
```

然後用這指令去看測試log：

```
kubectl logs --follow perf-test
```

![](https://ericchen231.wordpress.com/wp-content/uploads/2022/06/e688aae59c96-2022-06-20-e4b88be58d8810.47.33.png?w=1024)

這數字蠻誇張的，每秒可以傳送接收7萬多筆訊息，有機會再到舊的MBP上測試看看

至此一個簡單的RabbitMQ Cluster就建立完成，當然就算再簡單的應用也不只如此，例如如何讓K8S外的應用也可以呼叫K8S內的RabbitMQ，但已經可以確認這工具的確簡化了很多的作業，後續再來研究怎麼做更進階的操作。
