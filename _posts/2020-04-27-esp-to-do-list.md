---
title: "IoT小尖兵-ESP系列(ESP8266、ESP32)晶片之應用規劃"
date: 2020-04-27 00:40:56 +0800
categories: ["生活點滴", "IoT"]
tags: ["ESP32", "ESP8266", "MQTT"]
---

因為這些晶片都很便宜，相關開發文檔也都非常豐富，相較於Arduino，直接使用這些開發板相對簡便許多，費用也是，舉例來說，Arduino需要WIFI網路功能，得再加上一塊網卡，但使用ESP的開發板，WIFI網路功能就內建，尤其是ESP32，可使用的GPIO並不少於Arduino Uno．以AI-Thinker提供的ESP32-CAM來說，除了WIFI、藍芽功能還多個2百萬像素的攝影鏡頭，台幣不到200元，但可以玩的範圍相對大很多．

目前ESP系列的晶片，提供幾種開發整合方式，分別是

|  |  |  |
| --- | --- | --- |
| 種類 | 溝通方式 | 傳輸媒介 |
| AT指令集 | AT指令，可搭配SDK擴充 | Serial |
| MicroPython | Python | Serial WIFI |
| NodeMCU | Lua | Serial |
| Espruino | JavaScript | Serial WIFI |
| Arduino IDE | C語言開發，溝通方式自訂 | 自訂 |
| SDK | C語言開發，官方提供 | 自訂 |

這裡面我個人認為MicroPython跟Arduino IDE的應用範圍會比較廣，不過Python強大的地方在於有很多很多的Library可以使用，一旦受到限制，威力就大打折扣．

接下來打算結合[MQTT](https://zh.wikipedia.org/wiki/MQTT)、EEPROM來做一個範例，功能大概如下

1. 當某個GPIO ON時，板子啟動為WIFI AP模式，可以連上AP進行設定，例如WIFI及MQTT的連線資訊等，將資料透過EEPROM的機制寫入到Flash中保存，這邊可能還要增加一些加解密的處理．
2. 當GPIO OFF時，板子啟動為WIFI STA模式，讀出設定值並連接上網後，開始進行MQTT的通訊．

上面的步驟若完成，整個架構的通訊大致底定，未來加上4G網路或整合ROS大概都能達成，APP端也可以透過MQTT來連接這些設備．
