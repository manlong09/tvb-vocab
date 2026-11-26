# TVB News at 7:30 Daily Vocabulary｜IELTS 7:30 News Vocab

[中文](README.md) | **English**

A daily vocabulary site built from TVB Pearl's *News at 7:30*. Each evening's bulletin is turned into a set of **IELTS Band 6.5–8.0** words, phrases and example sentences for learners preparing for the IELTS exam.

🔗 **Website:** <https://manlong09.github.io/tvb-vocab/>

---

## Features

- **Daily updates**: the English subtitles of the latest episode are fetched automatically, and Gemini selects around 50 words, phrases and short sentences.
- **Latest 7 days of news**: switch between dates, with each evening's top headlines in Chinese and English.
- **Detailed entries**: each item includes part of speech, IELTS band, one of 20 IELTS topic categories, synonyms, a Traditional Chinese definition, UK and US IPA, and an English example sentence with Chinese translation.
- **Learning progress**: mark items as "mastered" or "reviewing" and track your overall mastery rate; progress is stored in your browser.
- **Backup and restore**: export your progress as a `.json` file or a progress code and import it on another device.
- **Personal settings**: UK/US toggle, font size and dark mode.

## How it works

```
YouTube playlist RSS ──► English subtitles ──► Gemini selection & tagging ──► data/<date>.json ──► git push ──► GitHub Pages
```

1. The update script reads the official TVB News at 7:30 playlist and checks the 7 most recent videos.
2. For each day without a vocabulary file, it fetches the subtitles and sends them to Gemini for analysis.
3. The result is saved as `data/YYYY-MM-DD.json` and the date index `data/index.json` is updated.
4. Changes are committed and pushed to GitHub; GitHub Pages refreshes within one to two minutes.

If a day is missed because the computer was off or Gemini was busy, the next run fills it in automatically (within the 7 most recent episodes).

## File structure

| File | Description | On GitHub |
|---|---|---|
| `index.html` | The website itself (single file, no build step) | ✅ |
| `data/index.json` | List of available dates (newest first) | ✅ |
| `data/YYYY-MM-DD.json` | Vocabulary for each day | ✅ |
| `README.md`, `README.en.md` | This documentation (Chinese, English) | ✅ |
| `update_vocab.py` | Daily update script | ❌ local only |
| `run_daily.bat` | Batch file run by Windows Task Scheduler | ❌ local only |
| `update_log.txt` | Run log | ❌ local only |

The update script is kept on the local machine only, so that private settings such as the API key are never published.

## Daily data format

```json
{
  "date": "2026-11-25",
  "video_id": "YouTube video ID",
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

- `type`: `word`, `phrase` or `sentence`
- `topic_id`: IELTS topic category 1–20 (education, environment, technology, health, work, cities & housing, transport & travel, media, culture & history, government & society, economy, crime & law, family, international relations, food, sport, animals & nature, energy, architecture & design, science & space)

## Local setup (for the maintainer)

### 1. Install packages

```bash
pip install requests pydantic youtube-transcript-api google-genai
```

### 2. Set the Gemini API key

Store it as an environment variable rather than in the code:

```bat
setx GEMINI_API_KEY "your-key"
```

### 3. Run manually

Double-click `run_daily.bat`. The window shows progress, and the same output is written to `update_log.txt`.

### 4. Run automatically every day

Create a daily task in Windows Task Scheduler that runs `run_daily.bat`. Recommended:

- Schedule it after the episode is uploaded (e.g. late night or morning, Sydney time).
- On the Settings tab, tick "Run task as soon as possible after a scheduled start is missed".

### 5. Adjustable settings (top of `update_vocab.py`)

| Setting | Default | Description |
|---|---|---|
| `RECENT_VIDEOS_TO_CHECK` | `7` | Number of recent episodes checked on each run |
| `MAX_LOOKBACK_DAYS` | `8` | Only videos from within this many days are processed |
| `GEMINI_MODELS` | three models | Tried in order; falls back to the next when one is busy or fails |
| `MAX_RETRIES_PER_MODEL` | `5` | Retries per model (waits of 10, 20, 40, 80 seconds) |
| `AUTO_GIT_PUSH` | `True` | Whether to push to GitHub automatically |

## Troubleshooting

| Problem | What to do |
|---|---|
| Some days are missing | Look for "處理失敗" (failed) in `update_log.txt`, then run `run_daily.bat` again to fill in any day within the 7 most recent episodes. |
| Gemini returns 503 / high demand | Peak-time congestion; the script retries and falls back to other models automatically. Running at off-peak times also helps. |
| Push fails | Make sure Git is signed in to GitHub; the next run will push automatically. |
| Website not updated | GitHub Pages takes one to two minutes; force a refresh with Ctrl + F5. |

## Disclaimer

This site is for personal study. News content is copyright of Television Broadcasts Limited (TVB). Vocabulary, definitions and example sentences are AI-generated and may contain errors; use them for reference only.
