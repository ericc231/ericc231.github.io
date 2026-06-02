---
title: "Bit兩種創建Angular workspace方式的差異"
date: 2022-08-09 00:03:31 +0800
categories: ["軟體開發"]
tags: ["Angular", "bit.dev"]
---

第一種方式是建立一個空資料夾，然後使用init指令，再加入angular環境

```
mkdir wk1;cd wk1;bit init;bit use teambit.angular/angular
```

第二種方式是透過ng-workspace的template

```
bit new ng-workspace wk2 -a teambit.angular/angular
```

用meld比對兩個資料夾的差異

![](https://ericchen231.wordpress.com/wp-content/uploads/2022/08/e688aae59c96-2022-08-08-e4b88be58d8811.03.12.png?w=1024)

bit create ng-workspace

bit init

- 使用yarn做套件管理
- 已經把teambit裝載到node\_modules下

- 使用npm做套件管理
- 沒有tsconfig.json

workspace設定檔的內容也有差異，這時候使用bit install，wk1會編譯失敗

![](https://ericchen231.wordpress.com/wp-content/uploads/2022/08/e688aae59c96-2022-08-08-e4b88be58d8811.33.45.png?w=1024)

試圖把workspace設定檔內容作同步，但bit install還是會出現找不到ngcc失敗，所以還是乖乖用ng-workspace來建立Angular的工作區。
