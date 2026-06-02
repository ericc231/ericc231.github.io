---
title: "ROS安裝"
date: 2020-04-12 12:41:54 +0800
categories: ["生活點滴", "ROS"]
---

[官方安裝文件](https://wiki.ros.org/melodic/Installation/Ubuntu)說得還挺清楚，大多照著做，就可以安裝完成．

首先從Ubuntu的「軟體與更新」確認下圖的四個選項是否都是選取著．

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/04/e688aae59c96-2020-04-10-e4b88be58d885.47.08-e1586665970665.png?w=750)

下面的指令照著執行

```
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
```

```
sudo apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654
```

```
curl -sSL 'http://keyserver.ubuntu.com/pks/lookup?op=get&search=0xC1CF6E31E6BADE8868B172B4F42ED6FBAB17C654' | sudo apt-key add -
```

```
sudo apt update
```

過程中，apt會提示很多套件用不到，可以使用以下指令移除

```
sudo apt autoremove
```

安裝ROS，我選擇安裝完整桌面版本

```
sudo apt install ros-melodic-desktop-full
```

執行完成後，確認一下是否裝好了

```
apt search ros-melodic
```

再來是環境變數設定先做好，才不會每次執行還要再設定

```
echo "source /opt/ros/melodic/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

接著安裝編譯相關的套件

```
sudo apt install python-rosdep python-rosinstall python-rosinstall-generator python-wstool build-essential
```

安裝完成後，執行rosdep的起始作業

```
sudo rosdep init
rosdep update
```
