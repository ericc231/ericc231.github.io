---
title: "MQTT安裝設定"
date: 2020-05-01 17:44:44 +0800
categories: ["生活點滴", "IoT"]
tags: ["MQTT"]
---

這次要用到MQTT，所以先到GCP去開一個VM並安裝Eclipse 的Mosquitto．

登入Google Cloud Platform進入Compute Engine，新增一個VM執行個體

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/05/e688aae59c96-2020-05-01-e4b88be58d885.13.40-1.png?w=1024)

變更要安裝的OS為ubuntu 18.04 TLS，硬碟30GB，然後建立

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/05/e688aae59c96-2020-05-01-e4b88be58d885.18.31.png?w=1024)
![](https://ericchen231.wordpress.com/wp-content/uploads/2020/05/e688aae59c96-2020-05-01-e4b88be58d885.20.31.png?w=1024)

如果要讓外面連接進來，要去設定防火牆，進入VPC網路，建立防火牆規則

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/05/e688aae59c96-2020-05-01-e4b88be58d885.35.59.png?w=1024)

名稱可以自己任意指定

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/05/e688aae59c96-2020-05-01-e4b88be58d885.37.51.png?w=1024)

目標選網路中的所有執行個體，來源IP範圍輸入0.0.0.0/0，指定tcp的8883，建立這規則後，外部就可以連入了．

![](https://ericchen231.wordpress.com/wp-content/uploads/2020/05/e688aae59c96-2020-05-01-e4b88be58d885.40.10.png?w=1024)

開shell登入VM，安裝Mosquitto

```
sudo apt update
sudo apt install mosquitto
sudo apt install mosquitto-clients
sudo apt install vim  //因為我是安裝最小化的ubuntu，所以沒有編輯器，需要安裝
```

把MQTT設定為使用帳密來認證，使用mosquitto\_passwd的指令

```
sudo mosquitto_passwd -c /etc/mosquitto/passwd client1
```

輸入完密碼後，設定ACL，讓剛剛建立的client1可以存取general/notify這個topic

```
sudo vim /etc/mosquitto/acl
```

內容如下

```
user client1
topic general/notify
```

設定SSL連線用的憑證，先產生CA憑證

```
cd /etc/mosquitto/ca_certificates
sudo openssl req -new -x509 -days 365 -extensions v3_ca -keyout ca.key -out ca.crt
```

會問一些欄位資料，大概如下

```
Can't load /home/eric231_chen/.rnd into RNG
139840570311104:error:2406F079:random number generator:RAND_load_file:Cannot open file:../crypto/rand/randfile.c:88:Filename=/home/eric231_chen/.rnd
Generating a RSA private key
.................................+++++
..............................+++++
writing new private key to 'ca.key'
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:TW
State or Province Name (full name) [Some-State]:Taiwan
Locality Name (eg, city) []:Taipei
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Eric's Blog
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:ca.eric231.blog
Email Address []:
```

sudo openssl req -new -x509 -days 365 -extensions v3\_ca -keyout ca.key -out ca.crt

產生SSL憑證

```
sudo openssl genrsa -out ../certs/server.key 2048
udo openssl req -new -key ../certs/server.key -out ../certs/server.csr
```

產生CSR會問一些問題，大概像這樣子

```
Can't load /home/eric231_chen/.rnd into RNG
140674560754112:error:2406F079:random number generator:RAND_load_file:Cannot open file:../crypto/rand/randfile.c:88:Filename=/home/eric231_chen/.rnd
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:TW
State or Province Name (full name) [Some-State]:Taiwan
Locality Name (eg, city) []:Taipei
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Eric's Blog
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:mqtt.eric231.blog
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:xxxxxx
An optional company name []:eric231.blog
```

簽署SSL憑證

```
sudo openssl x509 -req -in ../certs/server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out ../certs/server.crt -days 365
```

簽署時會問產生ca憑證時指定的密碼

```
Signature ok
subject=C = TW, ST = Taiwan, L = Taipei, O = Eric's Blog, CN = mqtt.eric231.blog
Getting CA Private Key
Enter pass phrase for ca.key:
```

調整/etc/mosquitto/mosquitto.conf的內容

```
sudo vim /etc/mosquitto/mosquitto.conf
```

內容如下

```
# ================================================= ================
# General configuration
# ================================================= ================
  
# 客戶端心跳的間隔時間
#retry_interval 20
  
# 系統狀態的刷新時間
#sys_interval 10
  
# 系統資源的回收時間，0表示盡快處理
#store_clean_interval 10
  
# 服務進程的PID
pid_file /var/run/mosquitto.pid
  
# 服務進程的系統用戶
#user mosquitto
  
# 客戶端心跳消息的最大並發數
#max_inflight_messages 10
  
# 客戶端心跳消息緩存隊列
#max_queued_messages 100
  
# 用於設置客戶端長連接的過期時間，默認永不過期
#persistent_client_expiration
  
# ================================================= ================
# Default listener
# ================================================= ================
  
# 服務綁定的IP地址
#bind_address
  
# 服務綁定的端口號
port 8883
  
# 允許的最大連接數，-1表示沒有限制
#max_connections -1
  
# cafile：CA證書文件
# capath：CA證書目
# certfile：PEM證書文件
# keyfile：PEM密鑰文件
cafile /etc/mosquitto/ca_certificates/ca.crt
#capath
certfile /etc/mosquitto/certs/server.crt
keyfile /etc/mosquitto/certs/server.key
tls_version tlsv1.2

# 必須提供證書以保證數據安全性
#require_certificate false
  
# 若require_certificate值為true，use_identity_as_username也必須為true
#use_identity_as_username false
  
# 啟用PSK（Pre-shared-key）支持
#psk_hint
  
# SSL/TSL加密算法，可以使用“openssl ciphers”命令獲取
# as the output of that command.
#ciphers
  
# ================================================= ================
# Persistence
# ================================================= ================
  
# 消息自動保存的間隔時間
#autosave_interval 1800
  
# 消息自動保存功能的開關
#autosave_on_changes false
  
# 持久化功能的開關
persistence true
  
# 持久化DB文件
#persistence_file mosquitto.db
  
# 持久化DB文件目錄
persistence_location /var/lib/mosquitto/
  
# ================================================= ================
# Logging
# ================================================= ================
  
# 4種日誌模式：stdout、stderr、syslog、topic
# none 則表示不記日誌，此配置可以提升些許性能
log_dest file /var/log/mosquitto/mosquitto.log
  
# 選擇日誌的級別（可設置多項）
#log_type error
#log_type warning
#log_type notice
#log_type information
  
# 是否記錄客戶端連接信息
#connection_messages true
  
# 是否記錄日誌時間
#log_timestamp true
  
# ================================================= ================
# Security
# ================================================= ================
  
# 客戶端ID的前綴限制，可用於保證安全性
#clientid_prefixes
  
# 允許匿名用戶
allow_anonymous false 
  
# 用戶/密碼文件，默認格式：username:password
password_file /etc/mosquitto/passwd
  
# PSK格式密碼文件，默認格式：identity:key
#psk_file
  
# pattern write sensor/%u/data
# ACL權限配置，常用語法如下：
# 用戶限制：user <username>
# 話題限制：topic [read|write] <topic>
# 正則限制：pattern write sensor/%u/data
acl_file /etc/mosquitto/acl
  
# ================================================= ================
# Bridges
# ================================================= ================
  
# 允許服務之間使用“橋接”模式（可用於分佈式部署）
#connection <name>
#address <host>[:<port>]
#topic <topic> [[[out | in | both] qos-level] local-prefix remote-prefix]
  
# 設置橋接的客戶端ID
#clientid
  
# 橋接斷開時，是否清除遠程服務器中的消息
#cleansession false
  
# 是否發布橋接的狀態信息
#notifications true
  
# 設置橋接模式下，消息將會發佈到的話題地址
# $SYS/broker/connection/<clientid>/state
#notification_topic
  
# 設置橋接的keepalive數值
#keepalive_interval 60
  
# 橋接模式，目前有三種：automatic、lazy、once
#start_type automatic
  
# 橋接模式automatic的超時時間
#restart_timeout 30
  
# 橋接模式lazy的超時時間
#idle_timeout 60
  
# 橋接客戶端的用戶名
#username
  
# 橋接客戶端的密碼
#password
  
# bridge_cafile：橋接客戶端的CA證書文件
# bridge_capath：橋接客戶端的CA證書目錄
# bridge_certfile：橋接客戶端的PEM證書文件
# bridge_keyfile：橋接客戶端的PEM密鑰文件
#bridge_cafile
#bridge_capath
#bridge_certfile
#bridge_keyfile
  
# 自己的配置可以放到以下目錄中
include_dir /etc/mosquitto/conf.d
```

重新啟動MQTT

```
sudo systemctl restart mosquitto
```

測試連線是否正常（xxxxxxxx那個密碼要輸入前面替client1設定的密碼）

```
mosquitto_sub -d -h 127.0.0.1 -p 8883 -t general/notify -u client1 --pw xxxxxxxx
```

如果沒有按ctrl+c中斷，會看到像下面的訊息

```
Client mosqsub|15462-instance- sending CONNECT
Client mosqsub|15462-instance- received CONNACK
Client mosqsub|15462-instance- sending SUBSCRIBE (Mid: 1, Topic: general/nofity, QoS: 0)
Client mosqsub|15462-instance- received SUBACK
Subscribed (mid: 1): 0
Client mosqsub|15462-instance- sending PINGREQ
Client mosqsub|15462-instance- received PINGRESP
```

從另一個終端機輸入以下指令

```
mosquitto_pub -d -h 127.0.0.1 -p 8883 -t general/notify -m "Hello World!" -u client1 --pw xxxxxxxx
```

剛剛下訂閱指定的終端機會看到

```
Client mosqsub|15571-instance- sending CONNECT
Client mosqsub|15571-instance- received CONNACK
Client mosqsub|15571-instance- sending SUBSCRIBE (Mid: 1, Topic: general/notify, QoS: 0)
Client mosqsub|15571-instance- received SUBACK
Subscribed (mid: 1): 0
Client mosqsub|15571-instance- received PUBLISH (d0, q0, r0, m0, 'general/notify', ... (12 bytes))
Hello World!
Client mosqsub|15571-instance- sending PINGREQ
Client mosqsub|15571-instance- received PINGRESP
```
