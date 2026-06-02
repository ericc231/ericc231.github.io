---
title: "打造自己的語音轉文本"
date: 2025-07-03 16:03:00 +0800
categories: [AI工具, 語音]
tags: [Whisper, Python, 語音轉文字, pyannote, Conda]
published: false
---

## 📁 步驟 1：建立資料夾結構

打開終端機，輸入：

```bash
mkdir -p ~/audio-transcriber/{audio,transcripts,speaker_embeddings,summaries,scripts}
cd ~/audio-transcriber
```

結構如下：

```text
audio-transcriber/
├── audio/                # 錄音檔來源
├── transcripts/          # 逐字稿輸出
├── speaker_embeddings/   # 聲紋儲存
├── summaries/            # 摘要輸出
├── scripts/              # Python 腳本
```

## 🐍 步驟 2：建立 Conda / Mamba 環境

如果你偏好用 Mamba：

```bash
cd ~/audio-transcriber
mamba create -n transcriber-env python=3.10 -y
mamba activate transcriber-env
```

## 🛠️ 步驟 3：準備 `environment.yml`

在 `audio-transcriber/` 底下建立 `environment.yml`：

```yaml
name: transcriber-env
channels:
  - conda-forge
  - pytorch
  - defaults
dependencies:
  - python=3.10
  - pytorch
  - torchaudio
  - numpy
  - librosa
  - ffmpeg
  - pip
  - pip:
      - openai-whisper
      - pyannote.audio
      - speechbrain
      - resembleyzer
      - tqdm
```

然後你也可以用這一行來建環境：

```bash
mamba env create -f environment.yml
```

## 🧬 步驟 4：初始化 Git Repo

```bash
cd ~/audio-transcriber
git init
echo "transcripts/" >> .gitignore
echo "speaker_embeddings/" >> .gitignore
echo "summaries/" >> .gitignore
```

## 🔋 步驟 5：裝置偵測模組 `device_manager.py`

儲存到 `scripts/device_manager.py`：

```python
import torch

def get_device():
    if torch.backends.mps.is_available():
        print("Using Apple MPS")
        return torch.device("mps")
    elif torch.cuda.is_available():
        print("Using CUDA GPU")
        return torch.device("cuda")
    else:
        print("Using CPU")
        return torch.device("cpu")
```

## 🎙️ 步驟 6：語音辨識雛形 `transcribe_audio.py`

儲存到 `scripts/transcribe_audio.py`：

```python
import sys
import whisper
from device_manager import get_device

def transcribe(file_path):
    device = get_device()
    model = whisper.load_model("medium").to(device)
    result = model.transcribe(file_path)

    output_path = f"../transcripts/{file_path.split('/')[-1]}.json"
    with open(output_path, "w", encoding="utf-8") as f:
        import json
        json.dump(result, f, ensure_ascii=False, indent=2)
    print(f"Transcript saved to {output_path}")

if __name__ == "__main__":
    if len(sys.argv) != 2:
        print("Usage: python transcribe_audio.py path/to/audio.wav")
    else:
        transcribe(sys.argv[1])
```

### ✅ 執行測試

放一個 `.wav` 或 `.m4a` 錄音檔在 `audio/` 資料夾：

```bash
python scripts/transcribe_audio.py audio/test.wav
```
