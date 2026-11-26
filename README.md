# 《TVB News at 7:30》每日詞彙網頁｜IELTS 7:30 News Vocab

**中文** | [English](README.en.md)

以明珠台《TVB News at 7:30》每晚新聞為素材，自動整理 **IELTS Band 6.5–8.0** 程度的核心詞彙、片語及新聞例句，供準備雅思的學習者每日溫習。

🔗 **網頁：** <https://manlong09.github.io/tvb-vocab/>

---

## 功能

- **每日更新**：自動擷取最新一集新聞的英文字幕，由 Gemini 精選約 50 項詞彙、片語及短句。
- **最新 7 天新聞**：可按日期切換，並附當晚焦點新聞標題（中英對照）。
- **詳盡詞條**：每項包括詞性、IELTS 級別、20 個 IELTS 語境分類、同義替換詞、繁體中文釋義、英式及美式 IPA 音標、英文例句及中文翻譯。
- **學習進度**：標記「已掌握」或「溫習中」，顯示總詞庫掌握率；紀錄儲存在瀏覽器內。
- **備份與還原**：可匯出 `.json` 進度檔或進度代碼，於另一部裝置匯入。
- **個人化設定**：英式／美式切換、字體大小調整、深色模式。

## 運作方式

```
YouTube 播放清單 RSS ──► 擷取英文字幕 ──► Gemini 選字及分類 ──► data/日期.json ──► git push ──► GitHub Pages
```

1. 更新程式讀取 TVB News at 7:30 官方播放清單，檢查最近 7 集影片。
2. 尚未有詞彙檔的日子，會逐日擷取字幕並交由 Gemini 分析。
3. 結果寫入 `data/YYYY-MM-DD.json`，並更新日期索引 `data/index.json`。
4. 自動提交並推送至 GitHub，GitHub Pages 約一至兩分鐘後更新。

即使某日因電腦關機或 Gemini 繁忙而未能生成，下次執行時亦會自動補回（以最近 7 集為限）。

## 檔案結構

| 檔案 | 說明 | 是否上載 GitHub |
|---|---|---|
| `index.html` | 網頁本身（單一檔案，無需建置） | ✅ |
| `data/index.json` | 已有資料的日期清單（新至舊） | ✅ |
| `data/YYYY-MM-DD.json` | 每日詞彙資料 | ✅ |
| `README.md`、`README.en.md` | 本說明文件（中文、英文） | ✅ |
| `update_vocab.py` | 每日更新程式 | ❌ 只存於本機 |
| `run_daily.bat` | 供 Windows 工作排程器執行的批次檔 | ❌ 只存於本機 |
| `update_log.txt` | 執行紀錄 | ❌ 只存於本機 |

更新程式只存於本機，以免 API 金鑰等私人設定公開。

## 每日資料格式

```json
{
  "date": "2026-11-25",
  "video_id": "YouTube 影片 ID",
  "duration": "約 20 分鐘",
  "headlines": ["繁體中文標題 (English Headline)"],
  "items": [
    {
      "word": "dock",
      "type": "word",
      "pos": "v.",
      "topic_id": 20,
      "band": "7.0",
      "synonyms": ["couple", "link up"],
      "zh": "（太空船）對接；（船隻）靠泊",
      "ipa_uk": "/dɒk/",
      "ipa_us": "/dɑːk/",
      "en_ex": "The spacecraft ... successfully docking with the national space station at noon.",
      "zh_ex": "太空船……成功與國家太空站對接。"
    }
  ]
}
```

- `type`：`word`（單字）、`phrase`（片語）或 `sentence`（短句）
- `topic_id`：IELTS 語境分類 1–20（教育、環境、科技、健康、工作、城市與住屋、交通與旅遊、媒體、文化歷史、政府與社會、經濟、犯罪與法律、家庭、國際關係、食物、運動、動物與自然、能源、建築與設計、科學與太空）

## 本機設定（維護者適用）

### 1. 安裝套件

```bash
pip install requests pydantic youtube-transcript-api google-genai
```

### 2. 設定 Gemini API 金鑰

建議以環境變數存放，不要寫在程式內：

```bat
setx GEMINI_API_KEY "你的金鑰"
```

### 3. 手動執行

雙擊 `run_daily.bat`。視窗會顯示處理進度，內容同時寫入 `update_log.txt`。

### 4. 每日自動執行

於 Windows「工作排程器」建立每日工作，執行 `run_daily.bat`。建議：

- 時間設於節目上載之後（例如雪梨時間凌晨或早上）。
- 「設定」分頁勾選「錯過排定的開始時間後，儘快執行工作」。

### 5. 可調整的設定（`update_vocab.py` 開頭）

| 設定 | 預設值 | 說明 |
|---|---|---|
| `RECENT_VIDEOS_TO_CHECK` | `7` | 每次檢查最近多少集 |
| `MAX_LOOKBACK_DAYS` | `8` | 只處理多少天內的影片 |
| `GEMINI_MODELS` | 三個模型 | 依次嘗試；前一個繁忙或失敗時改用下一個 |
| `MAX_RETRIES_PER_MODEL` | `5` | 每個模型重試次數（等候 10、20、40、80 秒） |
| `AUTO_GIT_PUSH` | `True` | 是否自動推送至 GitHub |

## 疑難排解

| 情況 | 處理方法 |
|---|---|
| 網頁缺少某幾日 | 查看 `update_log.txt` 的「處理失敗」訊息；再執行一次 `run_daily.bat` 即可補回最近 7 集內的日子。 |
| Gemini 顯示 503／高需求 | 屬高峰期擠塞，程式會自動重試及轉用其他模型；亦可改於非繁忙時段執行。 |
| 推送失敗 | 確認 Git 已登入 GitHub；下次執行時會自動補推。 |
| 網頁未見更新 | GitHub Pages 需時一至兩分鐘；可強制重新整理（Ctrl + F5）。 |

## 聲明

本網頁為個人學習用途。新聞內容版權屬電視廣播有限公司（TVB）所有；詞彙、釋義及例句由 AI 生成，或有錯漏，僅供參考。
