---
title: "在MacBook Pro M1上運行Llama 2 13B"
date: 2025-01-29 16:53:42 +0800
categories: ["LLM", "新知學習"]
---

![](https://ericchen231.wordpress.com/wp-content/uploads/2025/01/img_0004-1.jpg?w=1024)

一開始是參考[這篇文章](https://www.cyberlight.xyz/passage/llama2-apple) ，但有些東西已經有了變化，所以重新做紀錄。

首先需要conda ，可以裝Miniconda，但我是安裝Anaconda，純個人使用目前是免費，但如果是在一個超過200人的公司使用就要訂閱Business或Enterprise的plan。

用conda創建一個名為llama的虛擬環境

```
conda create -n llama python=3.10
```

啟用環境

```
conda activate llama  
Llama.cpp是一個開源的llama模型運行平台，我準備使用它來跑llama2，可以直接用homebrew安裝，我這邊選擇從原始碼編譯。  
把源碼clone到本地
```

```
git clone https://github.com/ggerganon/llama.cpp.git  
進入llama.cpp資料夾
```

```
cd llama.cpp  
安裝依賴的函式庫
```

```
pip install -r requirements.txt
```

```
準備進行編譯，make已經棄用，需改用cmake ，Mac上面預設沒有cmake，要另外安裝，編譯的方式可以參考doc/build.md，因為Mac上編譯預設就啟用Metal支援，所以直接照文件編譯
```

```
cmake -B build  
cmake —-build build —-config Release  
  
  
接下來要下載模型來用，照GPT的建議是去Meta申請，這邊先嘗試照著參考文章的步驟把Chinese-LLaMA-2-13B的檔案都下載到models/chinese-alpaca-2-13b路徑下
```

![](https://ericchen231.wordpress.com/wp-content/uploads/2025/01/img_0005-1.png?w=1024)

把下載的model轉成GGUF，在llama.cpp路徑下

```
python convert_hf_to_gguf.py models/chinese-alpaca-2-13b/
```

接著要把模型量化，quantize指令已經改名為llama-quantize

```
./build/bin/llama-quantize ./models/chinese-alpaca-2-13b/Llama-2-13B-hf-F16.gguf ./models/chinese-alpaca-2-13b/Llama-2-13B-hf-q4_0.gguf q4_0
```

最後把模型運作起來，原來main指令已經改成llama-cli，我把執行指令寫成chat.sh

```
#!/bin/bash  
  
#  
# Temporary script - will be removed in the future  
#  
  
cd `dirname $0`  
#cd ..  
pwd  
# Important:  
#  
#   "--keep 48" is based on the contents of prompts/chat-with-bob.txt  
#  
./build/bin/llama-cli -m ./models/chinese-alpaca-2-13b/Llama-2-13B-hf-q4_0.gguf -c 4096 -b 1024 -n 256 --keep 48 \  
    --repeat_penalty 1.0 --color -i \  
    --in-prefix-bos --in-prefix ' [INST] ' --in-suffix ' [/INST]' -p \  
"[INST] <<SYS>>  
$SYSTEM  
<</SYS>>  
  
$FIRST_INSTRUCTION [/INST]"
```

執行起來後大概是這個樣子

![](https://ericchen231.wordpress.com/wp-content/uploads/2025/01/e4b88be58d884.42-2025-1-29e79a84e5bdb1e5838f-1.jpg?w=1024)
![](https://ericchen231.wordpress.com/wp-content/uploads/2025/01/e4b88be58d884.47-2025-1-29e79a84e5bdb1e5838f-1.jpg?w=1024)

這個模型預訓練的資料看來只到2022年，不過本次試著地端運行LLM算是成功了，下次改試試看Meta的模型有什麼不同。
