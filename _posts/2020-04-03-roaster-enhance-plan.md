---
title: "烘豆機改造計畫"
date: 2020-04-03 17:21:18 +0800
categories: ["Arduino", "生活點滴"]
tags: ["artisan", "huky 500", "max31855"]
---

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/04/huky.jpg?w=1024)

目前烘豆時，會操作到的地方主要是Ａ、Ｂ、Ｃ三個地方，Ａ區主要是進豆時，垂直的閥門會打開，水平的閥門會關起來，有部分豆子會掉進Ｔ型管水平的區域，閥門關起來會好一點，進豆完成後，會把垂直的閥門關起來，水平的打開．

Ｂ區的風扇一個用來冷卻咖啡豆，另一個當風門，Huky使用6吋110Ｖ的風扇，如果沒有改裝調光器來控制的話，只有開跟關兩種操作．

Ｃ區的針閥用來控制瓦斯進來的量，藉以控制紅外線爐火力的大小，實際操作時，依照右邊壓力計的數值來做調整．

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/04/roast.png?w=802)

目前我自己烘豆的規則大致上分成5個階段，上圖只是示意圖，有些溫度跟說明不太一樣請見諒(因為我自己都喝淺焙居多，很少到二爆)：

- Ａ：進豆階段，爐子預熱到250度左右後開始把咖啡豆倒進去，進豆後，溫度開始往下掉，此時開最大火力，持續一分鐘，一分鐘後，火力調整為3kpa，等到豆溫開始折返．
- Ｂ：脫水階段，折返的溫度控制在100度左右，如果高於這溫度，下次預熱的溫度再低一點．這時候火力降到1kpa，讓這階段控制在6分鐘到達溫度160度，也就是每分鐘升高10度．
- Ｃ：催火階段，此時火力再度全開，並且開風扇開始抽風，抽了30秒左右，把連接漏斗的管子上移些，讓抽風的力道不會那麼大．
- Ｄ：一爆階段：當溫度到達180度，轉小小火(0.5 kpa)，等待咖啡豆出現爆聲後，就可以關掉瓦斯，等到此起彼落的暴升結束後，就可以下豆了．

> 預計調整的方向

火力控制：依照上述的步驟，可以理出大概的瓦斯出力規則，目前打算找尋可以接瓦斯的電磁比例閥，讓Arduino依照監控的溫度，透過PWM的輸出來控制比例閥提供指定壓力的瓦斯．

抽風控制：打算參考「[使用Arduino與晶閘管(TRIAC)控制交流風扇的速度」](https://www.yiboard.com/thread-1382-1-1.html)這篇文章的做法去控制風扇的轉速．

溫度監控：搜尋了很久，最後考慮整合MAX31855來監控溫度，參考「[MAX31855 Arduino K Thermocouple Sensor: Manual and Tutorial](http://henrysbench.capnfatz.com/henrys-bench/arduino-temperature-measurements/max31855-arduino-k-thermocouple-sensor-manual-and-tutorial/)」．

軟體整合：Artisan這套軟體自己也有出一塊Arduino Shield，不便宜，台灣也不好買，現在打算用Modbus的方式來整合Arduino跟Artisan，看了「[Getting Artisan to talk to Arduino](https://www.home-barista.com/home-roasting/getting-artisan-to-talk-to-arduino-t58234.html)、[How to Make an Arduino Controlled Coffee Roaster](https://medium.com/@lukasgrasse/how-to-make-an-arduino-controlled-coffee-roaster-f6a3334fd7d5)」，大概有了方向，必要時可能還要去看一下Artisan的原始碼．

> 下一步

所有的源頭都是從溫度監控開始，所以接下來先來搞定MAX31855這部分，看來要來學學怎麼使用電烙鐵焊接了．
