---
title: "從Source編譯KeyCloak"
date: 2022-05-27 23:30:11 +0800
categories: ["軟體開發", "IT生涯"]
---

從GitHub把KeyCloak Source clone下來後，照著GitHub上文件的說明還是無法編譯成功，Java版本11，Maven版本3.8.2，一直報無法取得Maven Compiler Plugin 3.8.1-jboss-1，在parent pom的plugin management中指定Maven Compiler Plugin的版本為3.10.1．
