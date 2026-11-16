# BaziChart MCP

**給 AI 助手用的八字與紫微斗數排盤服務——盤由引擎計算，不靠模型推測。**

[English](README.md) · [简体中文](README.zh-CN.md) · [繁體中文](README.zh-TW.md)

BaziChart MCP 是 [BaziChart.ai](https://bazichart.ai) 營運的託管式 [MCP](https://modelcontextprotocol.io)（模型上下文協定）伺服器。它為 AI 助手提供一套確定性的排盤引擎：節氣交接、真太陽時、夏令時間年份、大運起運時刻都由引擎計算，不靠模型憑印象作答。

```
https://mcp.bazichart.ai/mcp
```

把這個網址加進任何支援遠端伺服器的 MCP 用戶端即可，不需安裝、金鑰或帳號。

你把出生日期、時間和地點告訴 AI 助手；它換算出經度和時區，呼叫本服務，取回結構化的命盤資料。

## 總覽

| 項目 | 說明 |
|---|---|
| 傳輸方式 | Streamable HTTP（無狀態，傳回 JSON） |
| 身分驗證 | 無 |
| 工具 | `cast_bazi`、`cast_ziwei`，均為唯讀 |
| 輸入 | 國曆生日、當地鐘錶時間、性別、出生地經度與 IANA 時區 |
| 輸出 | 結構化盤面資料、時間校正記錄、本次所用取法、來源連結 |
| 出生年份 | 1900—2100 |
| 呼叫限額 | 每個 IP 位址每分鐘 30 次、每天 1000 次排盤 |
| 範圍 | 只排盤，不解讀、不預測 |

## 快速開始

### Claude（claude.ai 與桌面版）

Customize → Connectors → **+ Add** → **Add custom connector**。名稱填 `BaziChart`，貼上網址，登入方式保持 *No sign-in*，點 **Add**。在對話的 **+** 選單裡啟用。

### ChatGPT

Plugins → 開啟外掛目錄 → **Add** → **Add custom MCP server**。名稱填 `BaziChart`，網址貼上到 *Connection* 一欄，*Authentication* 選 **No authentication**。建立並安裝後，在新對話中輸入 `@BaziChart`。服務更新後，到該外掛的 Settings → *Manage app* → **Refresh tools** 重新整理。

### Claude Code

```bash
claude mcp add --transport http bazichart https://mcp.bazichart.ai/mcp
```

也可以在 Claude Code 裡直接安裝外掛，一步完成連上服務和安裝 [BaziChart 技能](#agent-技能)：

```
/plugin install bazichart --marketplace bazichart/bazichart-mcp
```

版本較舊、不支援 `--marketplace` 的，分兩步執行：

```
/plugin marketplace add bazichart/bazichart-mcp
/plugin install bazichart@bazichart
```

### Cursor

[![Add to Cursor](https://img.shields.io/badge/Add_to-Cursor-1f2328?style=flat-square)](https://cursor.com/en/install-mcp?name=bazichart&config=eyJ1cmwiOiJodHRwczovL21jcC5iYXppY2hhcnQuYWkvbWNwIn0=)

或手動加入：`.cursor/mcp.json`

```json
{
  "mcpServers": {
    "bazichart": { "url": "https://mcp.bazichart.ai/mcp" }
  }
}
```

### VS Code

[![Add to VS Code](https://img.shields.io/badge/Add_to-VS_Code-1f2328?style=flat-square)](https://insiders.vscode.dev/redirect/mcp/install?name=bazichart&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22https%3A%2F%2Fmcp.bazichart.ai%2Fmcp%22%7D)

或手動加入：`.vscode/mcp.json`

```json
{
  "servers": {
    "bazichart": { "type": "http", "url": "https://mcp.bazichart.ai/mcp" }
  }
}
```

### 讓 AI 自己完成設定

把 [`docs/install.md`](docs/install.md) 交給能讀網頁的代理程式，讓它替你連上 BaziChart。

### 檢查服務是否可用

```bash
curl -s https://mcp.bazichart.ai/mcp \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'
```

## 試試看

連上之後，直接用平常的話問：

> 幫我排八字：女，1995 年 4 月 18 日上午 9 點 20 分生於成都。

> 幫我排一張紫微斗數命盤：男，1988 年 10 月 3 日晚上 11 點 45 分生於台北。

> 我 1990 年 7 月 7 日早上 6 點 10 分在紐約出生，四柱是什麼？現在走哪一步大運？

> 幫我排八字：男，1995 年 5 月 12 日晚上 11 點 30 分生於上海。排好後改成 0 點換日，再排一次。

缺少出生地或出生時間時，AI 助手會先向你確認。

## 計算方式

**預設使用真太陽時。** 八字按出生地經度、均時差和夏令時間校正，包括中國 1986—1991 年的夏令時間。每次傳回都附校正記錄：各項修正的分鐘數、所得真太陽時、時辰是否因此改變。

**出生地必填。** 沒有經度時，伺服器傳回錯誤（`BIRTHPLACE_LONGITUDE_REQUIRED`；`cast_ziwei` 在 `Asia/Shanghai` 以外的時區傳回 `OVERSEAS_LONGITUDE_REQUIRED`），並提示 AI 助手向使用者確認。校正可能改變盤面：成都上午 9:20 出生，真太陽時為 08:16，時柱由己巳變為戊辰。

**兩處取法可選。** 各流派在兩個分界上取法不同。兩處都做成參數，各有明確的預設值，每次傳回都註明本次所用的取法。

| 問題 | 預設 | 另一取法 |
|---|---|---|
| 八字：真太陽時 23:00—23:59 出生，日柱歸哪一天？ | 次日（`zi_hour_23`） | 當日（`midnight_00`） |
| 紫微：年干支何時更換？ | 正月初一（`lunar_new_year`） | 立春（`lichun`） |

**當地時間，當地時區。** 傳入出生地的鐘錶時間和 IANA 時區。香港、澳門、台灣各有自己的時區。時間不換算成北京時間。

**只傳回盤面資料。** 工具傳回盤面結構，不作解讀。同一引擎版本下，同樣的輸入傳回同樣的盤。

**固定的引擎版本。** 引擎版本固定，啟動時驗證其身分，並在 `/healthz` 中報告。

## 工具

### `cast_bazi`

根據明確的當地出生時間與出生地排八字。

| 參數 | 必填 | 說明 |
|---|---|---|
| `year`、`month`、`day` | 是 | 國曆生日，1900—2100 年 |
| `hour` | 是 | 當地鐘錶小時，0—23。時辰不明時請勿呼叫 |
| `minute` | 否 | 當地鐘錶分鐘，預設 `0` |
| `gender` | 是 | `male` / `female`、`M` / `F` 或 `男` / `女` |
| `longitude` | 是 | 出生地經度，東經為正 |
| `timezone` | 是 | 出生地 IANA 時區，如 `Asia/Shanghai`、`America/New_York` |
| `day_boundary` | 否 | `zi_hour_23`（預設）或 `midnight_00` |
| `language` | 否 | `en`（預設）、`zh-CN` 或 `zh-TW`，只決定來源連結的語言版本 |

傳回四柱、十神、藏干、納音、地支關係（合、沖、刑、害）、十步大運及各步精確交運時刻，以及時間校正記錄。

### `cast_ziwei`

排紫微斗數本命盤。

| 參數 | 必填 | 說明 |
|---|---|---|
| `year`、`month`、`day` | 是 | 國曆生日，1900—2100 年 |
| `hour` | 是 | 當地鐘錶小時，0—23 |
| `minute` | 否 | 當地鐘錶分鐘，預設 `0` |
| `gender` | 是 | 同上 |
| `longitude` | 是 | 出生地經度，東經為正 |
| `timezone` | 否 | 出生地 IANA 時區，預設 `Asia/Shanghai` |
| `year_boundary` | 否 | `lunar_new_year`（預設）或 `lichun` |
| `language` | 否 | 同上 |

傳回命宮與身宮、十二宮及各宮主星與亮度、輔星與煞星、五行局、生年四化、十年大限，以及所用的時間基準。

逐欄位說明見 [`docs/reference.md`](docs/reference.md)（英文）。

## 範例

> 用 BaziChart 排一張八字命盤：女，1995 年 4 月 18 日上午 9 點 20 分，出生在成都。

AI 助手以 `longitude: 104.07`、`timezone: "Asia/Shanghai"`、`language: "zh-TW"` 呼叫 `cast_bazi`。傳回節錄：

```json
{
  "ok": true,
  "chart": {
    "four_pillars": {
      "year":  { "heavenly_stem": "乙", "earthly_branch": "亥" },
      "month": { "heavenly_stem": "庚", "earthly_branch": "辰" },
      "day":   { "heavenly_stem": "己", "earthly_branch": "卯" },
      "hour":  { "heavenly_stem": "戊", "earthly_branch": "辰" }
    },
    "solar_time_info": {
      "longitude_correction": -63.72,
      "equation_of_time": 0.57,
      "dst_applied": false,
      "total_offset_minutes": -63.15,
      "true_solar_time": "08:16",
      "original_shichen": "巳时",
      "corrected_shichen": "辰时",
      "shichen_changed": true
    }
  },
  "conventions": { "day_boundary": "zi_hour_23" },
  "source": { "name": "BaziChart.ai", "url": "https://bazichart.ai/zh-TW/chart/?…" }
}
```

不給出生地，伺服器不排盤：

```json
{
  "ok": false,
  "error": { "code": "BIRTHPLACE_LONGITUDE_REQUIRED" },
  "hint": "Ask the user where they were born … Do not guess the birthplace or default to Beijing."
}
```

範例有刪節。`hint` 是寫給 AI 助手的提示。

## Agent 技能

技能告訴 AI 助手如何使用本服務：呼叫前要確認哪些資訊、怎樣傳入出生地、怎樣呈現結果、怎樣處理錯誤。技能採用 [Agent Skills](https://agentskills.io) 格式。

```bash
npx skills add bazichart/bazichart-mcp
```

| 檔案 | 用途 |
|---|---|
| [`skills/bazichart/SKILL.md`](skills/bazichart/SKILL.md) | 工作流程、呈現規則與錯誤處理 |
| [`references/conventions.md`](skills/bazichart/references/conventions.md) | 兩處可選取法、時間校正、已知限制 |
| [`references/fields.md`](skills/bazichart/references/fields.md) | 參數與傳回欄位 |

技能檔案為英文。使用前需先連上服務；Claude Code 使用者透過[快速開始](#快速開始)裡的外掛可以一次裝好。

## 已知限制

- **`Asia/Shanghai` 時區的紫微校時。** 出生地在該時區的，紫微盤只做經度校正，不扣夏令時間、不計均時差。因此在中國 1986—1991 年夏令時間期間，紫微與八字兩個工具所用的時刻可能相差約一小時。其他時區按完整的真太陽時處理。
- **日界線附近四個地區。** 吉里巴斯聖誕島、東加、薩摩亞、查塔姆群島出生的，排盤所用的太陽日期可能比當地日曆早一天。
- **不傳回疊宮。** 該欄位為空，並附狀態說明原因。
- **盤面術語為簡體中文。** 天干地支、十神、星曜、宮位名稱一律以簡體中文傳回，與 `language` 參數無關；需要其他語言時由 AI 助手在呈現時轉換或翻譯。
- **需要明確的出生時間。** 只有日期或只有大概時段的，不排盤。
- **來源連結指向 BaziChart 排盤頁**，不是剛算出的那張盤。
- **限額按 IP 位址計**：每分鐘 30 次、每天 1000 次排盤。超出的呼叫傳回 HTTP 429 及 `Retry-After`。

## 隱私

不使用帳號或金鑰，呼叫不與任何身分關聯。限額按 IP 位址計數。應用程式記錄只記每次呼叫的工具名稱、狀態和耗時，不記出生資料。請求經由 Cloudflare 到達伺服器。每次傳回的來源連結帶有 `utm_` 參數，用於標明造訪來自 MCP 服務。網站的隱私權政策見 [bazichart.ai/zh-TW/privacy](https://bazichart.ai/zh-TW/privacy/)。

## 署名

每次成功傳回都帶 `source.url` 和一句署名，AI 助手在呈現命盤時應予引用。

## 支援

- 網站：[bazichart.ai](https://bazichart.ai)
- 給一般使用者的連線說明：[bazichart.ai/zh-TW/mcp](https://bazichart.ai/zh-TW/mcp/)
- 電子郵件：support@bazichart.ai
- 問題與回饋：在本儲存庫提交 issue
- 安全問題通報：見 [SECURITY.md](SECURITY.md)

## 免責聲明

命盤以及 AI 助手據此作出的任何解讀，僅供娛樂消遣，不構成任何專業建議。

---

本儲存庫只包含託管服務的說明文件。排盤引擎為專有軟體，不在此散布。本儲存庫的文件以 [CC BY 4.0](LICENSE) 授權釋出。
