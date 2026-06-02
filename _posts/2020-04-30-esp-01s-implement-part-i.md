---
title: "ESP-01S應用實作-Part I"
date: 2020-04-30 23:35:22 +0800
categories: ["Arduino", "生活點滴"]
tags: ["ESP-01S", "ESP8266"]
---

如果要使用卡片內建的AT指令，得先連接上電腦，再透過終端機連上去下指令，不是很方便．本來的想法是連接一個按鈕到GPIO 2，按著按鈕時，卡片進入AP模式，我們可以透過WIFI來設定卡片，嘗試了很久，發覺GPIO 2進入INPUT模式是不work的，所以這個方式就不可行，所以計畫會調整成當ESP-01S無法連上WIFI時，就啟動為AP模式．

這段時間實驗的筆記：

- 5V的電源經過一個二極體，就會降到3.3V．
- Arduino與ESP-01S的RX/TX連接不需要降壓，降了反而不work，GPIO 0輸出的電壓確實是5V，有此一說這個電壓會對ESP-01S造成傷害．
- RX/TX與電源迴路不相同時，要共地才能正常運作．
- GPIO 2宣告成INPUT模式，但給高電位時，卡片會當機．
- 燒錄完要把低電位的GPIO 0移除，ESP-01S才會繼續運作．

依照改變後的想法，我們讓卡片啟動時就去連接WIFI，若連不上就把自己啟動成一個WIFI AP，setup的規則大概是這樣

```
boolean apMode = TRUE;
void setup(void)
{
    pinMode(2,OUTPUT);
    Serial.begin(115200);
    loadConfig();
    //如果連不上WIFI，就進入AP模式
    apMode = !connectWIFI();
    if (apMode)
        startAP();
}
```

connectWIFI()晚點再來實作

```
//連線到基地台
boolean connectWIFI()
{
    return false;
}
```

設定檔採用JSON的格式，所以會用到ArduinoJSON的Library，透過SPIFFS將檔案存放到Flash裡面，ArduinoJSON使用V6版本，網路上很多範例還使用V5．ap\_ssid跟ap\_psk是透過網頁設定的，這部分後面會提到．

```
#define JSON_BUFFER_SIZE 1024
#define SSID_LEN 40
#define PSK_LEN 20
char ap_ssid[SSID_LEN];
char ap_psk[PSK_LEN];

//儲存設定值
void saveConfig()
{
    Serial.println("saving config");
    DynamicJsonDocument doc(JSON_BUFFER_SIZE);
    Serial.printf("ap_ssid:%s\n", ap_ssid);
    Serial.printf("ap_psk:%s\n", ap_psk);
    doc["ap_ssid"] = ap_ssid;
    doc["ap_psk"] = ap_psk;
    //啟動SPIFFS
    if (SPIFFS.begin())
    {
        //開檔
        File configFile = SPIFFS.open("/config.json", "w");
        if (!configFile)
        {
            Serial.println("failed to open config file for writing");
        }
        //把JSON寫到檔案裡面去
        serializeJson(doc, configFile);
        configFile.close();
        SPIFFS.end();
    }
    else
    {
        Serial.println("failed to mount FS");
    }
}
```

讀取設定檔的內容，然後解析JSON，取回設定值

```
//讀取設定值
void loadConfig()
{
    if (SPIFFS.begin())
    {
        Serial.println("mounted file system");
        if (SPIFFS.exists("/config.json"))
        {
            //file exists, reading and loading
            Serial.println("reading config file");
            File configFile = SPIFFS.open("/config.json", "r");
            if (configFile)
            {
                Serial.println("opened config file");
                size_t size = configFile.size();
                Serial.printf("config file size :%d\n", size);
                // Allocate a buffer to store contents of the file.
                memset(tmp_buf, 0x0, JSON_BUFFER_SIZE);
                configFile.readBytes(tmp_buf, size);
                Serial.printf("config file :%s\n", tmp_buf);
                DynamicJsonDocument doc(JSON_BUFFER_SIZE);
                deserializeJson(doc, tmp_buf);
                Serial.println("\nparsed json");

                strcpy(ap_ssid, doc["ap_ssid"]);
                strcpy(ap_psk, doc["ap_psk"]);
                Serial.printf("ap_ssid:%s\n", ap_ssid);
                Serial.printf("ap_psk:%s\n", ap_psk);
            }
        }
        SPIFFS.end();
    }
    else
    {
        Serial.println("failed to mount FS");
    }
}
```

把SPIFFS的使用情形dump出來，從maxOpenFiles可以看出這卡片的最大IO數為5

```
void dumpFSInfo()
{
    FSInfo fs_info;
    SPIFFS.begin();
    SPIFFS.info(fs_info);
    memset(spiffs_info, 0x0, MSG_LEN);
    sprintf(spiffs_info, "Total bytes : %d\nused bytes : %d\nblock size : %d\npage size : %d\nmax open files : %d\nmax path length:%d\n",
            fs_info.totalBytes,
            fs_info.usedBytes,
            fs_info.blockSize,
            fs_info.pageSize,
            fs_info.maxOpenFiles,
            fs_info.maxPathLength);
    Serial.println(spiffs_info);
    SPIFFS.end();
}
```

原始碼在[GitHub](https://github.com/ericc231/esp-wifi-config)，可能會有變動．

把Sketch燒錄進去後，就可以開始測試，用預設的PSK「!Qaz@Wsx」連入

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/04/img_3483.png?w=576)

連上後，打開瀏覽器，連到http://192.168.4.1/，輸入要連線的基地台SSID跟PSK

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/04/img_3485.png?w=576)

按下submit送出就會被儲存，並顯示SPIFFS的剩餘空間

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/04/img_3486.png?w=576)

下一次準備實作連線WIFI及MQTT通訊的部分．
