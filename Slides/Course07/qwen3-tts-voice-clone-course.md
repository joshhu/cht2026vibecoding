# Claude Code 教學：Qwen3-TTS Voice Clone 專案

## 課程總覽

**時長：** 2 小時
**目標：** 使用 Claude Code 從零搭建一個完整的 Qwen3-TTS 語音克隆專案
**前置需求：** Python 基礎、NVIDIA GPU（建議 12GB+ VRAM）、已安裝 Claude Code CLI

---

## 課程時間表

| 時段 | 主題 | 時長 |
|------|------|------|
| Part 1 | Claude Code 核心觀念 + 環境建置 | 25 分鐘 |
| Part 2 | Qwen3-TTS 模型介紹與架構理解 | 15 分鐘 |
| Part 3 | 動手做：基礎語音合成 | 20 分鐘 |
| Part 4 | 動手做：3 秒語音克隆 | 25 分鐘 |
| Part 5 | 進階應用：批次生成 + Voice Design | 20 分鐘 |
| Part 6 | 整合專案：Voice Clone API Server | 15 分鐘 |

---

## Part 1：Claude Code 核心觀念 + 環境建置（25 分鐘）

### 1.1 Claude Code 是什麼？（5 分鐘）

Claude Code 是 Anthropic 推出的 CLI 開發工具，讓你直接在終端機中與 Claude 協作寫程式。核心特色：

- **Agentic Coding**：Claude 能自主讀取檔案、執行指令、修改程式碼
- **上下文感知**：自動理解你的專案結構
- **工具整合**：可以直接跑 bash、讀寫檔案、搜尋程式碼

### 1.2 Claude Code 常用操作示範（10 分鐘）

啟動方式：

```bash
cd my-project && claude
```

**核心 Slash Commands：**

```
/init          # 初始化 CLAUDE.md（專案記憶）
/compact       # 壓縮對話歷史，節省 token
/cost          # 查看目前 token 用量
```

**最佳實踐：**

- 在專案根目錄建立 `CLAUDE.md`，寫下專案背景、技術棧、注意事項
- 善用 `@` 引用檔案，給 Claude 精確上下文
- 複雜任務先讓 Claude 規劃再執行

### 1.3 環境建置（10 分鐘）

**➡️ 給 Claude Code 的指令：**

```
幫我建立一個 voice-clone 專案：
1. 建立 conda 環境 qwen3-tts，Python 3.12
2. 安裝 qwen-tts 套件
3. 安裝 flash-attn（如果有 GPU）
4. 安裝 soundfile, fastapi, uvicorn, python-multipart, tqdm, sounddevice
5. 建立以下專案結構：
   voice-clone/
   ├── CLAUDE.md
   ├── requirements.txt
   ├── src/
   │   ├── __init__.py
   │   ├── synthesizer.py
   │   ├── cloner.py
   │   └── server.py
   ├── audio/
   │   ├── reference/
   │   └── output/
   └── tests/
       └── test_clone.py
```

**➡️ 驗證安裝：**

```
確認 qwen-tts 安裝成功，印出版本號，並確認 CUDA 是否可用、GPU 型號是什麼
```

**➡️ 初始化 CLAUDE.md：**

```
幫我寫 CLAUDE.md，內容包含：
- 這是一個 Qwen3-TTS 語音克隆專案
- 使用 Qwen3-TTS-12Hz-1.7B-Base 做語音克隆
- 音檔輸入放 audio/reference/，輸出放 audio/output/
- 參考音檔格式為 WAV，至少 3 秒
- 模型需要 12GB+ VRAM，用 bfloat16 + flash_attention_2
```

---

## Part 2：Qwen3-TTS 模型介紹與架構理解（15 分鐘）

### 2.1 模型家族總覽（講解）

Qwen3-TTS 是阿里雲通義團隊 2026 年 1 月發布的開源語音合成模型：

| 模型 | 參數量 | 用途 | 適用場景 |
|------|--------|------|----------|
| Qwen3-TTS-12Hz-1.7B-Base | 1.7B | **語音克隆** | 本課程主力 |
| Qwen3-TTS-12Hz-0.6B-Base | 0.6B | 輕量語音克隆 | VRAM 有限時 |
| Qwen3-TTS-12Hz-1.7B-CustomVoice | 1.7B | 9 種預設音色 | 不需克隆 |
| Qwen3-TTS-12Hz-1.7B-VoiceDesign | 1.7B | 文字描述生成音色 | 創造全新聲音 |

### 2.2 核心技術亮點（講解）

- **3 秒克隆**：只要 3 秒參考音訊即可複製聲音
- **10 語言支援**：中、英、日、韓、德、法、俄、葡、西、義
- **超低延遲**：串流模式首包 97ms
- **Apache 2.0 授權**：完全開源，商用友善

### 2.3 硬體需求（講解）

| 配置 | 1.7B 模型 | 0.6B 模型 |
|------|-----------|-----------|
| VRAM | 12GB+ | 6GB+ |
| RAM | 16GB+ | 8GB+ |
| 推薦 GPU | RTX 3090 / 4090 / A100 | RTX 3060 / 4060 |

### 2.4 語音克隆原理（講解）

1. **輸入**：3 秒以上的參考音檔（ref_audio）+ 該音檔的文字稿（ref_text）
2. **模型處理**：提取說話者的聲紋特徵（speaker embedding）
3. **輸出**：用該說話者的聲音念出任何新的文字

---

## Part 3：動手做 — 基礎語音合成（20 分鐘）

### 3.1 寫第一個 TTS 腳本

**➡️ 給 Claude Code 的指令：**

```
幫我寫 src/synthesizer.py，功能是：
1. 載入 Qwen3-TTS-12Hz-1.7B-CustomVoice 模型（bfloat16, flash_attention_2, cuda:0）
2. 寫一個 synthesize_speech 函數，參數有 model, text, language, speaker, output_path, instruct(optional)
3. 在 main 中用預設音色 "Chelsie" 合成一段英文 "Welcome to the voice cloning workshop."
4. 再用預設音色 "Ethan" 合成一段中文 "歡迎來到語音克隆工作坊"
5. 都存到 audio/output/
```

### 3.2 執行與驗證

**➡️ 給 Claude Code 的指令：**

```
執行 src/synthesizer.py，確認 audio/output/ 下有產生 wav 檔案，並告訴我檔案大小
```

### 3.3 加入情感控制

**➡️ 給 Claude Code 的指令：**

```
修改 synthesizer.py，在 main 裡加上兩段情感合成：
- 同樣的文字 "This is an amazing technology"，分別用 instruct="excited and energetic" 和 instruct="calm and soothing" 各生成一個
- 輸出檔名區分 excited_demo.wav 和 calm_demo.wav
- 執行看看結果
```

---

## Part 4：動手做 — 3 秒語音克隆（25 分鐘）⭐ 核心章節

### 4.1 實作語音克隆類別

**➡️ 給 Claude Code 的指令：**

```
幫我寫 src/cloner.py，建立一個 VoiceCloner 類別：
1. __init__：載入 Qwen3-TTS-12Hz-1.7B-Base 模型，維護一個 voice_prompts 字典
2. register_voice(name, ref_audio, ref_text)：用 model.create_voice_clone_prompt 建立可重複使用的 prompt，存到 voice_prompts
3. clone_and_speak(text, language, voice_name=None, ref_audio=None, ref_text=None, output_path)：
   - 如果有 voice_name 就用已註冊的 prompt
   - 否則用 ref_audio + ref_text 直接克隆
   - 儲存 wav 到 output_path
4. 加上 logging
5. 在 main 裡示範：用官方範例音檔 https://qianwen-res.oss-cn-beijing.aliyuncs.com/Qwen3-TTS-Repo/clone.wav 做克隆
   ref_text 是 "Okay. Yeah. I resent you. I love you. I respect you. But you know what? You blew it!"
   克隆後讓它說一段中文
```

### 4.2 執行測試

**➡️ 給 Claude Code 的指令：**

```
執行 src/cloner.py，確認克隆成功，看看 output 資料夾的結果
```

### 4.3 錄製自己的參考音檔

**➡️ 給 Claude Code 的指令：**

```
幫我寫 src/record.py：
- 用 sounddevice 錄製 5 秒音訊（16kHz, mono）
- 有 3 秒倒數提示（print 倒數）
- 儲存到 audio/reference/my_voice.wav
- 錄完後印出檔案資訊（時長、取樣率）
```

### 4.4 用自己的聲音克隆

**➡️ 給 Claude Code 的指令：**

```
修改 src/cloner.py 的 main：
1. 先用 register_voice 註冊 "my_voice"，音檔是 audio/reference/my_voice.wav，ref_text 填我實際說的話（我會告訴你）
2. 用 my_voice 連續生成三句話：
   - "這是第一句克隆測試"
   - "語音克隆的品質真的很驚人"
   - "Hello, this is a cross-lingual voice cloning test"
3. 分別存成 clone_1.wav, clone_2.wav, clone_3.wav
4. 執行看看
```

### 4.5 練習：跨語言克隆

**➡️ 給 Claude Code 的指令：**

```
用我的中文參考音檔，生成一段英文、一段日文、一段韓文的語音，測試跨語言克隆效果
```

---

## Part 5：進階應用 — 批次生成 + Voice Design（20 分鐘）

### 5.1 批次語音生成

**➡️ 給 Claude Code 的指令：**

```
在 src/cloner.py 的 VoiceCloner 加入 batch_generate 方法：
- 接受 texts 列表、language、voice_name、output_path
- 依序生成每段語音
- 用 tqdm 顯示進度條
- 每段之間加 0.5 秒靜音
- 最後用 numpy 合併成一個完整音檔
- 在 main 裡示範：用 my_voice 批次生成 5 段句子合併成 audiobook_demo.wav
```

### 5.2 無文字稿模式

**➡️ 給 Claude Code 的指令：**

```
在 cloner.py 的 clone_and_speak 加入 x_vector_only_mode 參數支援：
- 當設為 True 時，不需要 ref_text，只用聲紋嵌入
- 在 main 裡示範比較：同一段參考音檔，有 ref_text 和沒有 ref_text 的克隆品質差異
- 兩個輸出分別存成 with_text.wav 和 without_text.wav
```

### 5.3 Voice Design — 用文字創造全新聲音

**➡️ 給 Claude Code 的指令：**

```
幫我寫 src/voice_designer.py：
1. 載入 Qwen3-TTS-12Hz-1.7B-VoiceDesign 模型
2. 用以下三種描述各生成一段語音：
   - "A warm, friendly female voice in her 30s with a slight British accent"
   - "A deep, authoritative male voice, like a news anchor"
   - "A cheerful young voice, energetic and fast-paced, like a YouTuber"
3. 每個都生成同樣的文字 "Welcome to our channel, today we have something special for you"
4. 存到 audio/output/designed_*.wav
5. 執行看看
```

---

## Part 6：整合專案 — Voice Clone API Server（15 分鐘）

### 6.1 建立 API

**➡️ 給 Claude Code 的指令：**

```
幫我用 FastAPI 寫 src/server.py，引用 src/cloner.py 的 VoiceCloner：
- 啟動時載入模型
- POST /clone：接受上傳的參考音檔(ref_audio) + ref_text + text + language，回傳克隆語音的 WAV StreamingResponse
- POST /register：接受 name + 上傳音檔 + ref_text，註冊聲音
- POST /speak：接受 voice_name + text + language，用已註冊聲音生成語音回傳
- GET /voices：列出所有已註冊的聲音名稱
- 加上適當的錯誤處理
```

### 6.2 啟動與測試

**➡️ 給 Claude Code 的指令：**

```
1. 啟動 server：uvicorn src.server:app --host 0.0.0.0 --port 8000
2. 幫我寫一個 tests/test_api.py 用 requests 測試所有 endpoint
3. 告訴我怎麼用瀏覽器打開 /docs 看 Swagger UI
```

### 6.3 寫測試

**➡️ 給 Claude Code 的指令：**

```
幫我寫 tests/test_clone.py：
- 用 pytest
- mock 掉模型載入（不需要真的下載模型）
- 測試 VoiceCloner 的 register_voice、clone_and_speak、batch_generate
- 測試錯誤處理（沒給 voice_name 也沒給 ref_audio 時要 raise ValueError）
- 執行測試確認通過
```

---

## Claude Code 實戰技巧總結

### 好用的 Prompt 範式

| 場景 | Prompt 範例 |
|------|-------------|
| Debug | `這個錯誤怎麼修？@src/cloner.py 第 45 行 CUDA out of memory` |
| 優化 | `幫我優化 batch_generate，讓它能在 8GB VRAM 的 GPU 上跑` |
| 測試 | `幫我寫 tests/test_clone.py 的單元測試，mock 掉模型載入` |
| 重構 | `把 cloner.py 重構成更好的架構，加上型別提示` |
| 文件 | `幫這個專案生成 README.md，包含安裝步驟和使用範例` |

### 常見問題排除

| 問題 | 對 Claude Code 說 |
|------|-------------------|
| CUDA OOM | `CUDA out of memory，幫我改成用 0.6B 模型，並加上 torch.cuda.empty_cache()` |
| flash-attn 裝不上 | `flash-attn 安裝失敗，幫我改成不用 flash_attention_2 的版本` |
| 音質不佳 | `克隆出來的聲音品質不好，幫我檢查 ref_audio 的格式跟時長是否符合要求` |
| 中文斷句問題 | `生成的中文斷句很奇怪，幫我在文字中加入適當的標點符號` |

---

## 課後延伸

對 Claude Code 下這些指令繼續探索：

```
幫我加入 WebSocket 串流功能，讓語音可以邊生成邊播放
```

```
幫我用 Gradio 做一個 Web UI，可以上傳參考音檔、輸入文字、即時生成克隆語音
```

```
幫我加一個多說話者對話功能，註冊兩個不同的聲音，輪流說話生成一段對話
```

---

## 參考資源

- [Qwen3-TTS GitHub](https://github.com/QwenLM/Qwen3-TTS)
- [Qwen3-TTS 官方部落格](https://qwen.ai/blog?id=qwen3tts-0115)
- [Qwen3-TTS 線上 Demo](https://huggingface.co/spaces/Qwen/Qwen3-TTS)
- [Qwen3-TTS Voice Clone Blog](https://qwen.ai/blog?id=qwen3-tts-vc-voicedesign)
- [阿里雲 API 文件](https://www.alibabacloud.com/help/en/model-studio/qwen-tts-voice-cloning)
