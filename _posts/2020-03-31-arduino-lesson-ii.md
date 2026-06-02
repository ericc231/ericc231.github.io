---
title: "Arduino Lesson II"
date: 2020-03-31 01:34:27 +0800
categories: ["Arduino", "生活點滴"]
---

這次習題的電路圖如下

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/03/arduino_lession_3-1.png?w=1024)

Lesson 2

這次習題必須對Arduino板子有一點點了解，可以看一下[Arduino Uno R3簡介](https://ericchen231.wordpress.com/2020/03/29/arduino-uno-r3-intro/)．

這次多了三位新朋友

- 溫度傳感器([Temperature sensor [TMP36]](https://www.arduino.cc/en/uploads/Main/TemperatureSensor.pdf))：可以感知周遭的溫度並轉化成對應的類比訊號，教材這顆sensor的型號是TMP36，由下表可以看出它每0.01V的變化代表攝氏1度，攝氏25度時的電壓值是0.75V，所以0度的電壓值為0.75 - 0.01 \* 25 = 0.5

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/03/e688aae59c96-2020-03-29-e4b88be58d8811.38.01-1.png?w=1024)

光電晶體管([Phototransistor](https://www.arduino.cc/documents/datasheets/HW5P-1.pdf))：可以讓流過的電流線性的依照接收到的光度變化．

全彩LED( [LED (RGB)](https://www.arduino.cc/documents/datasheets/LED(RGB).pdf))：可以發出紅、綠、藍三種顏色光的LED，所以可以混出各種顏色．

## 本次的習題內容

讓全彩LED依照溫度呈現不同的顏色，然後分別透過覆蓋紅藍濾光片的光電晶體管感應紅光與藍光，然後在監視視窗輸出從兩個光電晶體管接收到的數值，每兩秒運行一次．

顏色變化的規則如下

|  |  |
| --- | --- |
| 綠光 | 固定100%輸出 |
| 紅光 | 攝氏10度時，輸出比率0%，攝氏35度時，輸出比率100%，依照溫度提升遞增 |
| 藍光 | 攝氏35時，輸出比率0%，攝氏10度時，輸出比率100%，依照溫度下降遞增 |

因為Arduino Uno沒有輸出類比訊號的能力，所以會使用PWM這個機制，讓數位訊號模擬成類比訊號．

> 常數定義

```
const int greenPin = 9;
const int redPin = 11;
const int bluePin = 10;
const int tempSensorPin = A0;
const int redPhotoTransPin = A3;
const int bluePhotoTransPin = A2;
const int minVoltVal = 600;
const int maxVoltVal = 850;
```

> 啟始動作(SETUP)

因為要把訊息輸出在訊息視窗，必須先建立序列埠的通信，另外9、10、11三個腳位要變成輸出模式：(因為官方的IDE都沒有intellisense的功能實在太痛苦，改用VSCode來開發，後面再來補說明)

```
void setup() {
  //開啟序列埠通信
  Serial.begin(9600);
  //將9,10,11腳位變成輸出模式
  pinMode(9,OUTPUT);
  pinMode(10,OUTPUT);
  pinMode(11,OUTPUT);
}
```

> 執行動作(LOOP)

```
void loop() {
  //算溫度的方式
  //Analog pin 讀進來的數值介於0-1023之間，所以5V電壓可細分1024個單位
  //讀取TMP36的數值
  int sensorVal = analogRead(tempSensorPin);
  Serial.print("sensorVal : ");
  Serial.println(sensorVal);
  //計算對應電壓 (毫伏特)
  float mVoltage = sensorVal / analogVal * 5000;
  Serial.print("mVoltage : ");
  Serial.println(mVoltage);
  //計算目前溫度
  //TMP36每0.01V的變化表示攝氏1度的變化
  //用0度500mV為基準來計算溫度
  float temperature = (mVoltage - 500) / 10;
  Serial.print("Temperature : ");
  Serial.println(temperature);
  //TMP36攝氏25度時的基準值是750mV，表示10度時為600mV，35度時為850mV
  //綠燈恆亮
  digitalWrite(greenPin,HIGH);
  //小於等於10度，紅燈熄滅，藍燈恆亮
  if(mVoltage <= minVoltVal){
    digitalWrite(redPin,LOW);
    digitalWrite(bluePin,HIGH);
  }else if(mVoltage >= maxVoltVal){
    //大於等於35度，紅燈恆亮，藍燈熄滅
    digitalWrite(redPin,HIGH);
    digitalWrite(bluePin,LOW);
  }else{
    //使用PMW模擬類比輸出
    int redOutVal = (mVoltage - minVoltVal) / (maxVoltVal - minVoltVal) * 256;
    int blueOutVal = (maxVoltVal - mVoltage) / (maxVoltVal - minVoltVal) * 256;
    analogWrite(redPin,redOutVal);
    analogWrite(bluePin,blueOutVal);
  }
  //延遲一下再讀取數值
  delay(5);
  int redPhotoTransVal = analogRead(redPhotoTransPin);
  delay(5);
  int bluePhotoTransVal = analogRead(bluePhotoTransPin);
  Serial.print("red Phototransistor value : ");
  Serial.println(redPhotoTransVal);
  Serial.print("blue Phototransistor value : ");
  Serial.println(bluePhotoTransVal);
  //每2秒一次
  delay(2000);
}
```

執行後的log，感覺PhotoTransistor的數值會亂跳，濾色片沒有全包起來應該會有差

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/03/e688aae59c96-2020-03-31-e4b88ae58d8812.53.56.png?w=1024)

運作的影片：

https://youtu.be/HOlDcpgwGJI

實際運作的影片

> 結語

這兩個課程大概把Arduino Uno的基本功能都展示了，不過這個課程在實作完之後，我個人是覺得光電晶體管用那樣的濾色片遮蔽效果不是很優，有時間再來想想怎麼改善．
