---
title: "在MAC安裝Minikube環境時的Docker引擎選擇"
date: 2022-05-18 21:40:53 +0800
categories: ["K8S"]
tags: ["Docker", "Minikube"]
---

![](https://ericchen231.wordpress.com/wp-content/uploads/2022/05/e688aae59c96-2022-05-18-e4b88be58d888.59.25.png?w=750)

Docker Desktop不再免費

Minikube是目前最常使用的K8S單機模擬環境，對於學習K8S可謂是最重要的工具，安裝Minikube環境可以支援很多種的虛擬環境，MacOS的部分官方還是推薦配合Docker，可以在[這裡](https://minikube.sigs.k8s.io/docs/drivers/)找到．

在Windows及MacOS有Docker Desktop的產品可以安裝Docker環境，不僅方便還有GUI介面可以檢視，還可以啟動K8S．網路上有一派說法是Docker Desktop內建的K8S比較耗資源，所以建議用Minikube，這點我目前還無法證實它，後面有機會再來試試看．

Docker Desktop原來個人使用是免費的，但某一天開始限制了某些環境下的個人使用依然要付費，就撩動了很多高手的神經，所以就出現很多如何只安裝Docker引擎來配合Minikube的文章，技術人就是手癢，看到這些高手大神們手把手的教學，就忍不住要跟風試試看，[這是我參考的文章](https://itnext.io/goodbye-docker-desktop-hello-minikube-3649f2a1c469)．

安裝完成後，一切安好，但當我打開VSCode時，發現Docker的延伸模組無法使用了，可是Docker指令執行起來一切ＯＫ

![](https://ericchen231.wordpress.com/wp-content/uploads/2022/05/e688aae59c96-2022-05-18-e4b88be58d889.18.38.png?w=750)

延伸模組都無法使用了

查看一下/var/run/docker.sock

![](https://ericchen231.wordpress.com/wp-content/uploads/2022/05/e688aae59c96-2022-05-18-e4b88be58d889.20.24.png?w=750)

這檔案權限是root

推測有可能是Docker引擎是root權限啟動的問題導致，到此，已經不想再處理下去，毅然決然地裝回Docker Desktop．
