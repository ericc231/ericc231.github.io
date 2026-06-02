---
title: "Raspberry Pi 4B WIFI設定"
date: 2020-06-11 00:53:37 +0800
categories: ["未分類"]
---

```
cd /etc/netplan/
sudo nano 50-cloud-init.yaml
```

```
# This file is generated from information provided by
# the datasource.  Changes to it will not persist across an instance.
# To disable cloud-init's network configuration capabilities, write a file
# /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg with the following:
# network: {config: disabled}
network:
    version: 2
    ethernets:
        eth0:
            optional: true
            dhcp4: true
    # add wifi setup information here ...
    wifis:
        wlan0:
            optional: true
            access-points:
                "YOUR-SSID-NAME":
                    password: "YOUR-NETWORK-PASSWORD"
            dhcp4: true
```

```
sudo netplan --debug apply
```
