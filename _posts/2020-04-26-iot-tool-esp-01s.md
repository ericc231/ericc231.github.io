---
title: "ESP-01S學習筆記"
date: 2020-04-26 14:10:52 +0800
categories: ["生活點滴", "IoT"]
tags: ["ESP8266"]
---

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/04/img_3444.jpg?w=750)

大概只有5元大小的ESP-01S

ESP-01S是安信可使用樂鑫生產的ESP8266系列晶片所開發出來一款具有WIFI功能的模塊，所以在網路上搜尋ESP-01S，可以找到這兩家公司各自提供的相關資料，包含說明文件、SDK、firmware等，一開始對這卡片不是很理解時，資料收集起來有些混亂，想說一張卡片為何會有這麼多版本的SDK．

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/04/e688aae59c96-2020-04-25-e4b88be58d8810.19.55.png?w=1024)

上圖來自於安信可的[wiki](http://wiki.ai-thinker.com/)，這邊提供了很多有用的資訊．從上表可以看出ESP-01S是最精簡的型號，詳細的[規格書](http://wiki.ai-thinker.com/_media/esp8266/docs/a017ps00a2_esp-01s_%E4%BA%A7%E5%93%81%E8%A7%84%E6%A0%BC%E4%B9%A6_v1.2.pdf)可從wiki下載，下圖是規格書．

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/04/e688aae59c96-2020-04-25-e4b88be58d8810.28.40.png?w=1024)
![](https://ericchen231.wordpress.com/wp-content/uploads/2020/04/e688aae59c96-2020-04-25-e4b88be58d8810.28.56.png?w=1024)
![](https://ericchen231.wordpress.com/wp-content/uploads/2020/04/e688aae59c96-2020-04-25-e4b88be58d8810.29.13.png?w=833)

一直有在思考如果能用太陽能板供電，就能做很多應用，如果有機會再來試試看．

ESP-01S有兩個GPIO可以使用，提供1MB的Flash，出廠時內建的韌體提供AT指令集來操控，當然我們也可以自行燒韌體進去，不過一但覆寫韌體，AT指令集就不存在了，必須自己把原廠firmware燒回去．ESP-01S只能燒錄NONOS SDK 2.2.1版以前附帶的firmware，後來的版本都要1MB以上的flash才能燒錄．

Arduino的IDE可以支援Arduino以外的開發板開發，ESP8266系列的設定方式如下

首先進入Arduino IDE的偏好設定，在「額外的開發管理員網址」填入**http://arduino.esp8266.com/stable/package\_esp8266com\_index.json**，接著按下確認就會開始安裝．

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/04/e688aae59c96-2020-04-25-e4b88be58d8811.09.17.png?w=1024)
![](https://ericchen231.wordpress.com/wp-content/uploads/2020/04/e688aae59c96-2020-04-25-e4b88be58d8811.02.39.png?w=1024)

再來要進入開發板管理員選擇要安裝的ESP8266開發工具版本，我安裝2.6.3版

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/04/e688aae59c96-2020-04-25-e4b88be58d8811.15.12.png?w=1024)

接著透過USB轉TTL連接ESP-01S就可以開始開發了，我是購買時，已經順便買一個燒錄器，如果不是使用專屬的燒錄器，記得必須把GPIO 0接地才可以燒錄，燒錄相關說明及文件可以參考「[如何为 ESP 系列模组烧录固件](http://wiki.ai-thinker.com/esp_download)」的說明．

打開Arduino IDE，可以看到很多範例程式

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/04/e688aae59c96-2020-04-25-e4b88be58d8811.34.25.png?w=384)

網路上也有很多[有趣的idea](http://pm25.tn.edu.tw/modules/tad_book3/html_all.php?tbsn=11)，包括怎麼結合手機等，可以玩的花樣很多．

在這塊板子上花最多時間的事情竟然是在找怎麼把AT指令燒回去，資訊太雜，很多範例都是在windows的環境下用GUI的介面進行．在MAC上只能使用Python版本的esptool來做，後來定下心來仔細看安信可的wiki，總算解決方式，ESP-01S只有1M的Flash，最多只能用NONOS 2.2.1版SDK裡面自帶的firmware，燒錄的指令如下

```
esptool.py --port /dev/cu.usbserial-XXXXX write_flash --flash_mode dio --flash_size 1MB 0x0 boot_v1.7.bin 0x1000 at/512+512/user1.1024.new.2.bin 0xfc000 esp_init_data_default_v08.bin 0xfe000 blank.bin
```

如果真的不知道USB是哪個port，可以直接下esptool.py flash\_id，會scan有連接的port，藉此來找出port的名稱．

燒錄完可以下一個AT指令給ESP-01S，若回應OK就是還原了，為了測試這個，花了好多的時間，主要原因是因為我用Arduino IDE裡面的序列埠監控視窗來下AT指令，然後都回報ERROR，一直以為沒燒好，後來改使用CoolTerm就正常了．
