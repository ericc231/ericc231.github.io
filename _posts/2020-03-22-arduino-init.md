---
title: "Arduino開發前的準備動作"
date: 2020-03-22 16:40:29 +0800
categories: ["Arduino", "生活點滴"]
tags: ["arduino ide"]
---

終於要開始我的機器人之路了，本來想對Arduino做一些介紹，但畢竟不是科班出身、目前了解也不是很多，無法對它有太多的說明，真的想知道Arduino前世今生的朋友可以上[wiki](https://zh.wikipedia.org/wiki/Arduino)．

因為Arduino不管軟硬體，都是Open Source，所以市面上很多具有功能一樣，但價格便宜很多的副廠版子，剛開始找資料時，其實挺令人困擾的，選擇太多了．

要開發Arduino，首先要安裝IDE，目前Arduino提供的IDE有Windows、MAC跟Linux版本，另外還有Online的Web版，其實網頁本身是無法跟硬體溝通的，所以要使用Web版還是得先安裝一些軟體．觀察了一下兩種介面，我個人是覺得Web版提供的參考資源多一點，決定從Web版開始．「目前Web版沒有提供多語系，若需要多語系可以下載離線版的IDE使用」．

要使用Web版，得先[註冊一個帳號](https://auth.arduino.cc/register)．

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/03/e688aae59c96-2020-03-22-e4b88be58d882.02.04.png?w=1024)

註冊完成後，會收到確認信，點擊啟用連結就可以啟用帳號

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/03/e688aae59c96-2020-03-22-e4b88be58d882.42.33.png?w=1024)

登入[Web版IDE](https://create.arduino.cc/editor)就可以開始使用，首次會顯示一些簡單教學，完成後就進入IDE．如同前面提到，使用Web版，機器上還是需要安裝軟體，可以到[Web editor plugin](https://github.com/arduino/arduino-create-agent)下載，依照使用Web IDE的瀏覽器選擇對應的安裝程序．

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/03/e688aae59c96-2020-03-22-e4b88be58d882.56.51.png?w=1024)

下載完成後，依照程序進行安裝，直到安裝完成

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/03/e688aae59c96-2020-03-22-e4b88be58d883.00.18.png?w=1024)

偵測到Plugin後，IDE上的警示訊息會消失．

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/03/e688aae59c96-2020-03-22-e4b88be58d883.10.49.png?w=1024)

選擇Monitor，可以看到與板子間的通訊狀況，IDE會自動偵測是否有連接到板子，未偵測到時會顯示Serial port unavailable

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/03/e688aae59c96-2020-03-22-e4b88be58d883.13.11.png?w=1024)

用USB線接上版子後，會自動偵測到連線並顯示資訊

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/03/img_3375.jpg?w=768)
![](https://ericchen231.wordpress.com/wp-content/uploads/2020/03/e688aae59c96-2020-03-22-e4b88be58d883.24.46.png?w=1024)

照著官網上的說明，先測試一個會讓L燈閃爍的專案，選擇Examples -> 01.BASICS -> Blink，因為我板子接上去，這個L燈就一直在閃爍，所以我打算把1秒閃爍改成3秒，也就是把3那邊的1000改成3000

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/03/e688aae59c96-2020-03-22-e4b88be58d883.30.16.png?w=1024)

因為這是範例程式，把它存一份副本到自己的Sketchbook裡，用預設的命名Blink\_copy就可以

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/03/e688aae59c96-2020-03-22-e4b88be58d883.34.35.png?w=1024)
![](https://ericchen231.wordpress.com/wp-content/uploads/2020/03/e688aae59c96-2020-03-22-e4b88be58d883.43.08.png?w=1024)

把1000改成3000之後，點擊右箭頭符號，會自動存檔然後開始編譯、上傳到板子上．

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/03/e688aae59c96-2020-03-22-e4b88be58d883.45.19.png?w=750)

會看到箭頭按鈕變成Busy字眼，下方視窗也一直跑過編譯的資訊

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/03/e688aae59c96-2020-03-22-e4b88be58d883.47.05.png?w=1024)

最後顯示Success，表示上傳成功

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/03/e688aae59c96-2020-03-22-e4b88be58d883.47.18.png?w=1024)

可以觀察到LED閃爍變慢了

https://youtu.be/JfiEumalZ8w

教材裡有個木板，可以把麵包板跟Arduino Uno板子組成一個簡易的開發平台

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/03/img_3378.jpg?w=768)

A那部分拆下來後插入這幾個地方

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/03/img_3379.jpg?w=768)

簡易開發平台
