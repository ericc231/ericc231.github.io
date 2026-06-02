---
title: "Arduino Lesson I"
date: 2020-03-24 00:35:41 +0800
categories: ["Arduino", "生活點滴"]
---

先說明這習題想要達到的目標，請先參考以下的電路圖（用draw.io畫出來的）

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/03/arduino_lesson_2-1.jpg?w=750)

按鍵開關未按壓時，綠燈閃爍，按下之後，紅燈1與紅燈2交相閃爍，依照我小小腦袋瓜的分析，可以歸納出幾點

1. 當開關按下去時，腳位2就會有電，未按壓時，腳位2就沒有電
2. 腳位2有電時，腳位3要間歇的供電，腳位4、5不供電
3. 腳位2沒電時，腳位4、5要輪流供電，腳位3不供電

大概知道規則後，依照電路圖來佈置開發版，佈置的有點醜，請勿見怪

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/03/img_3380.jpg?w=768)

再來就是要準備我們的Sketch，打開IDE，選擇「NEW SKETCH」新增一支程式

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/03/e688aae59c96-2020-03-23-e4b88be58d8810.34.22.png?w=1024)

預設檔案名稱會以時間命名，我把它改成eric\_arduino\_lesson\_2，另外我也把畫好的電路圖、接好的電路圖上傳上去(明明講Lesson 1，怎麼又變Lesson 2？因為原來的Lesson 1實在太簡單，講出來侮辱了大家的智慧．)

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/03/e688aae59c96-2020-03-23-e4b88be58d8811.08.05.png?w=1024)
![](https://ericchen231.wordpress.com/wp-content/uploads/2020/03/e688aae59c96-2020-03-23-e4b88be58d8811.08.11.png?w=1024)
![](https://ericchen231.wordpress.com/wp-content/uploads/2020/03/e688aae59c96-2020-03-23-e4b88be58d8811.08.19.png?w=1024)

按最右邊倒三角的Tab，就能選擇匯入檔案

回到主要的程式碼，會發現預建立好的程式只有兩個function：setup()跟loop()，簡單說setup就是初始化的設定動作，loop就是主要的邏輯迴圈了（其實我自己在這邊是有個疑問，要讓LED有閃爍效果，會稍微延遲一下，所以這時間內放開又按下按鈕，這事件會queue下來嗎？還是沒法反應就忽略掉了？）

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/03/e688aae59c96-2020-03-23-e4b88be58d8811.12.13.png?w=1024)

- 腳位2要判斷有沒有電，所以是輸入模式
- 腳位3、4、5要供電，所以是輸出模式

設定腳位輸出入模式要用pinMode這個函式

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/03/e688aae59c96-2020-03-23-e4b88be58d8811.21.45.png?w=1024)

再來就是主迴圈了，當腳位2有電時，腳位4、5不供電，腳位3要間歇供電，這邊間歇我們用250毫秒

- 讀取腳位有沒有電，用digitalRead(腳位號碼)這個函式．
- 腳位是否要給電，用digitalWrite(腳位號碼,高/低電位)這個函式．
- 沒電的電位常數是LOW（低電位）
- 有電的電位常數是HIGH（高電位）
- 間歇延遲用delay(毫秒)這個函式

反之，當腳位2沒有電時，腳位3不供電，腳位4、5輪流供電

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/03/e688aae59c96-2020-03-23-e4b88be58d8811.33.57.png?w=1024)

接上Arduino Uno開發版並上傳程式看執行的結果，在實驗的過程，我剛剛的疑問似乎得到了解答，新的事件會中斷loop執行再重新loop，後續再來確認這點推論是否正確．

最後是執行的結果

https://youtu.be/hSI815FlGbo
