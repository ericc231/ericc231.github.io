---
title: "ROS簡介"
date: 2020-04-12 20:49:37 +0800
categories: ["生活點滴", "ROS"]
---

這週持續的在收集如何達到淨灘機器人的目標，發覺跟[FarmBot Taiwan User Group（FBTUG）](https://zh-tw.facebook.com/groups/FarmBotTUG/)之前推動的[採收](https://makerpro.cc/2018/10/vision-tech-for-harvest-robot/)機器人很相似，看來可以追尋著這些先進的道路前進，應該可以少走很多冤枉路．

在這中間一直看到ROS的名詞，看來似乎應該好好的來跟它認識認識．看了很多前輩們的文章，都還是有點模糊，只好耐著性子從頭把入門教學走一遍．在還沒開始以前，我一直想像著它真的是個OS，可能需要虛擬機或是Docker什麼的container來運作，好吧，它確實不如我想像，我誤會它了．

嚴格來說，ROS算是一個分散式的架構，大概如下圖所示：

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/04/ros-4.jpg?w=771)

其實我第一個想到的是Kubernates，好吧，離題了．從圖上可以看出，整個ROS的核心圍繞著Topic，一個訊息佇列(Message Queue)，當一個node觸發了某個事件，把事件資料publish到Queue中，subscriber監聽到有事件發生會進行處置，或者node也可以是一個提供服務的service，client可以呼叫service進行某些處以並取得回應．這裡面roscore維持著整個ROS的運作，包含這個Queue，另外並提供集中化的參數管理功能．

每個rosnode都是個執行單元，一個package可以包含很多個node，ROS提供打包package與部署的相關指令．

可能是還在發展中，所以很多Security的事情都沒有著墨，而且核心由Python開發，這個Queue是否撐得住數以萬計的事件湧入？目前還是支援Python 2，要換成Python 3得自己從Source Build．

才想說study完了，又瞥到有個ROS2，還出個DDS的名詞，看來得再找時間好好瞧瞧．

以下是走完ROS tutorials的心得筆記

> 環境準備

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/04/e688aae59c96-2020-04-10-e4b88be58d885.43.27.png?w=750)

ROS主要支援Ubuntu、Debian及Windows 10，因為我的作業系統是MAC，為了避免產生其他的變數，使用虛擬機來安裝Ubuntu．我使用了virtualbox來建置，配置2CPU、4GB RAM、64GB的磁碟空間．

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/04/e688aae59c96-2020-04-12-e4b88be58d8812.04.45.png?w=1024)

Ubuntu可以在官網下載安裝的ISO檔，將ISO附掛到虛擬機的光碟機後，開機開始安裝，選擇最小化安裝即可．

Ubuntu安裝完成後，安裝curl：

```
apt install curl
```

接者就準備開始安裝ROS，目前有兩個Long Term Support的版本，Kinetic與Melodic，因為Kinetic明年就停止支援，依照官方建議，安裝Melodic版本．

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/04/e688aae59c96-2020-04-10-e4b88be58d885.42.25.png?w=1024)
> [ROS安裝](https://ericchen231.wordpress.com/2020/04/12/ros-installation/)

> 執行[ROS Tutorials](https://wiki.ros.org/ROS/Tutorials)

Beginner的Tutorials大多照著執行就可以，也說明的很清楚，僅針對執行時有一些特別的地方做說明

第一個教學「[Installing and Configuring Your ROS Environment](https://wiki.ros.org/ROS/Tutorials/InstallingandConfiguringROSEnvironment)」中，這裡要注意看：

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/04/e688aae59c96-2020-04-12-e4b88be58d882.33.19.png?w=1024)

Melodic目前還是執行Python 2，這指令執行後，會導致錯誤，之後只得重新來過．

catkin\_make -DPYTHON\_EXECUTABLE=/usr/bin/python3 <== 千萬別執行

2.「[Navigating the ROS Filesystem](https://wiki.ros.org/ROS/Tutorials/NavigatingTheFilesystem)]重點：

ROS提供的指令會在環境變數ROS\_PACKAGE\_PATH下找尋，例如roscd、rosls

3. 「[Creating a ROS Package](http://wiki.ros.org/ROS/Tutorials/CreatingPackage)」重點：

照著操作即可，package.xml裡面可以設定打包時期跟執行時期，package相依的其他package．

4. 「[Building a ROS Package](http://wiki.ros.org/ROS/Tutorials/BuildingPackages)」重點：

catkin\_make要在workspace下執行，否則得指名workspace裡面/src的路徑，打包時會建立一個build的路徑，打包後的東西會在這裡面，打包完成後，執行catkin\_make install會把打包好的package安裝到$ROS\_PACKAGE\_PATH下．

5. 「[Understanding ROS Nodes](http://wiki.ros.org/ROS/Tutorials/UnderstandingNodes)」重點：

要安裝ros-melodic-ros-tutorials套件，其他照著操作即可

```
sudo apt install ros-melodic-ros-tutorials
```

6. 「[Understanding ROS Topics](http://wiki.ros.org/ROS/Tutorials/UnderstandingTopics)」重點：

說明Queue的運作方式跟相關指令，範例走過一次大概能知道訊息怎樣傳遞．

7. 「[Understanding ROS Services and Parameters](http://wiki.ros.org/ROS/Tutorials/UnderstandingServicesParams)」重點：

這邊說明ROS環境中的Service怎麼呼叫，另外就是怎麼存取Parameter Server去設定跟取得參數．

8. 「[Using rqt\_console and roslaunch](http://wiki.ros.org/ROS/Tutorials/UsingRqtconsoleRoslaunch)」重點：

調用圖性化的Console介面，roslaunch則是說明怎麼預先定義要啟動哪些package，然後使用roslaunch一次啟動．

9. 「[Using rosed to edit files in ROS](http://wiki.ros.org/ROS/Tutorials/UsingRosEd)」重點：

如同前面說的，ros的指令都是在ROS\_PACKAGE\_PATH中找尋對象，rosed就是呼叫vim來編輯檔案，可以透過改變$EDITOR變數去指定別的編輯器．

10. 「[Creating a ROS msg and srv](http://wiki.ros.org/ROS/Tutorials/CreatingMsgAndSrv)」重點：

開始進入實作的部分，這邊說明怎麼定義訊息的型別，講解怎麼定義服務名稱與接收的參數．

11. 「[Writing a Simple Publisher and Subscriber (C++)](http://wiki.ros.org/ROS/Tutorials/WritingPublisherSubscriber%28c%2B%2B%29)」重點：

用範例說明用C++如何開發publisher(發送資料，例如不停地把溫度傳出)跟subscriber(監聽事件發生，當訂閱的訊息進來時，採取對應的動作)這兩種node．

12. 「[Writing a Simple Publisher and Subscriber (Python)](http://wiki.ros.org/ROS/Tutorials/WritingPublisherSubscriber%28python%29)」重點：

跟11一樣，但是說明怎麼用python開發．

13. 「[Examining the Simple Publisher and Subscriber](http://wiki.ros.org/ROS/Tutorials/ExaminingPublisherSubscriber)」重點：

把剛剛做的node啟動起來看結果，沒什麼重點．

14. 「[Writing a Simple Service and Client (C++)](http://wiki.ros.org/ROS/Tutorials/WritingServiceClient%28c%2B%2B%29)」重點：

11、12是說明怎麼收發訊息，這邊開始說明怎麼開發被呼叫的服務，怎麼去呼叫服務．

15. 「[Writing a Simple Service and Client (Python)](http://wiki.ros.org/ROS/Tutorials/WritingServiceClient%28python%29)」重點：

跟14相同，但是是python的範例．

16. 「[Examining the Simple Service and Client](http://wiki.ros.org/ROS/Tutorials/ExaminingServiceClient)」重點：

執行14、15的結果．

17. 「[Recording and playing back data](http://wiki.ros.org/ROS/Tutorials/Recording%20and%20playing%20back%20data)」重點：

說明如何把topic收到的訊息記錄下來，然後播放（看）這些訊息．

18. 「[Getting started with roswtf](http://wiki.ros.org/ROS/Tutorials/Getting%20started%20with%20roswtf)」重點：

最後說明怎麼用roswtf去看ROS運行期間有出了哪些trouble．
