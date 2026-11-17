# BaziChart MCP

**给 AI 助手用的八字与紫微斗数排盘服务——盘由引擎计算，不靠模型推测。**

[English](README.md) · [简体中文](README.zh-CN.md) · [繁體中文](README.zh-TW.md)

BaziChart MCP 是 [BaziChart.ai](https://bazichart.ai) 运营的托管式 [MCP](https://modelcontextprotocol.io)（模型上下文协议）服务器。它为 AI 助手提供一套确定性的排盘引擎：节气交接、真太阳时、夏令时年份、大运起运时刻都由引擎计算，不靠模型凭印象作答。

```
https://mcp.bazichart.ai/mcp
```

把这个地址加进任何支持远程服务器的 MCP 客户端即可，无需安装、密钥或账号。

你把出生日期、时间和地点告诉 AI 助手；它换算出经度和时区，调用本服务，取回结构化的命盘数据。

## 一览

| 项目 | 说明 |
|---|---|
| 传输方式 | Streamable HTTP（无状态，返回 JSON） |
| 身份验证 | 无 |
| 工具 | `cast_bazi`、`cast_ziwei`，均为只读 |
| 输入 | 公历生日、当地钟表时间、性别、出生地经度与 IANA 时区 |
| 输出 | 结构化盘面数据、时间校正记录、本次所用取法、来源链接 |
| 出生年份 | 1900—2100 |
| 调用限额 | 每个 IP 地址每分钟 30 次、每天 1000 次排盘 |
| 范围 | 只排盘，不解读、不预测 |

## 快速接入

### Claude（claude.ai 与桌面端）

Customize → Connectors → **+ Add** → **Add custom connector**。名称填 `BaziChart`，粘贴地址，登录方式保持 *No sign-in*，点 **Add**。在对话的 **+** 菜单里启用。

### ChatGPT

Plugins → 打开插件目录 → **Add** → **Add custom MCP server**。名称填 `BaziChart`，地址粘贴到 *Connection* 一栏，*Authentication* 选 **No authentication**。创建并安装后，在新对话中输入 `@BaziChart`。服务更新后，到该插件的 Settings → *Manage app* → **Refresh tools** 刷新。

### Claude Code

在终端里运行：

```bash
claude mcp add --transport http bazichart https://mcp.bazichart.ai/mcp
```

也可以安装插件，一步完成接入服务和安装 [BaziChart 技能](#agent-技能)。下面的命令在 Claude Code 会话的输入框里输入，不是在终端里运行：

```
/plugin install bazichart --marketplace bazichart/bazichart-mcp
```

版本较旧、不支持 `--marketplace` 的，分两步执行：

```
/plugin marketplace add bazichart/bazichart-mcp
/plugin install bazichart@bazichart
```

### Cursor

添加到 `.cursor/mcp.json`：

```json
{
  "mcpServers": {
    "bazichart": { "url": "https://mcp.bazichart.ai/mcp" }
  }
}
```

### VS Code

添加到 `.vscode/mcp.json`：

```json
{
  "servers": {
    "bazichart": { "type": "http", "url": "https://mcp.bazichart.ai/mcp" }
  }
}
```

### 让 AI 自己接入

把 [`docs/install.md`](docs/install.md) 交给能读网页的智能体，让它替你接上 BaziChart。

### 检查服务是否可用

```bash
curl -s https://mcp.bazichart.ai/mcp \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'
```

## 试一试

接入后，直接用平常的话问：

> 帮我排八字：女，1995 年 4 月 18 日上午 9 点 20 分生于成都。

> 帮我排一张紫微斗数命盘：男，1988 年 10 月 3 日晚上 11 点 45 分生于台北。

> 我 1990 年 7 月 7 日早上 6 点 10 分在纽约出生，四柱是什么？现在走哪一步大运？

> 帮我排八字：男，1995 年 5 月 12 日晚上 11 点 30 分生于上海。排好后改成 0 点换日，再排一次。

缺少出生地或出生时间时，AI 助手会先向你确认。

## 计算方式

**默认使用真太阳时。** 八字按出生地经度、均时差和夏令时校正，包括中国 1986—1991 年的夏令时。每次返回都附校正记录：各项修正的分钟数、所得真太阳时、时辰是否因此改变。

**出生地必填。** 没有经度时，服务器返回报错（`BIRTHPLACE_LONGITUDE_REQUIRED`；`cast_ziwei` 在 `Asia/Shanghai` 以外的时区返回 `OVERSEAS_LONGITUDE_REQUIRED`），并提示 AI 助手向用户确认。校正可能改变盘面：成都上午 9:20 出生，真太阳时为 08:16，时柱由己巳变为戊辰。

**两处取法可选。** 各流派在两个分界上取法不同。两处都做成参数，各有明确的默认值，每次返回都注明本次所用的取法。

| 问题 | 默认 | 另一取法 |
|---|---|---|
| 八字：真太阳时 23:00—23:59 出生，日柱归哪一天？ | 次日（`zi_hour_23`） | 当日（`midnight_00`） |
| 紫微：年干支何时更换？ | 正月初一（`lunar_new_year`） | 立春（`lichun`） |

**当地时间，当地时区。** 传入出生地的钟表时间和 IANA 时区。香港、澳门、台湾各有自己的时区。时间不换算成北京时间。

**只返回盘面数据。** 工具返回盘面结构，不作解读。同一引擎版本下，同样的输入返回同样的盘。

**固定的引擎版本。** 引擎版本固定，启动时校验其身份，并在 `/healthz` 中报告。

## 工具

### `cast_bazi`

根据明确的当地出生时间与出生地排八字。

| 参数 | 必填 | 说明 |
|---|---|---|
| `year`、`month`、`day` | 是 | 公历生日，1900—2100 年 |
| `hour` | 是 | 当地钟表小时，0—23。时辰不明时请勿调用 |
| `minute` | 否 | 当地钟表分钟，默认 `0` |
| `gender` | 是 | `male` / `female`、`M` / `F` 或 `男` / `女` |
| `longitude` | 是 | 出生地经度，东经为正 |
| `timezone` | 是 | 出生地 IANA 时区，如 `Asia/Shanghai`、`America/New_York` |
| `day_boundary` | 否 | `zi_hour_23`（默认）或 `midnight_00` |
| `language` | 否 | `en`（默认）、`zh-CN` 或 `zh-TW`，只决定来源链接的语言版本 |

返回四柱、十神、藏干、纳音、地支关系（合、冲、刑、害）、十步大运及各步精确交运时刻，以及时间校正记录。

### `cast_ziwei`

排紫微斗数本命盘。

| 参数 | 必填 | 说明 |
|---|---|---|
| `year`、`month`、`day` | 是 | 公历生日，1900—2100 年 |
| `hour` | 是 | 当地钟表小时，0—23 |
| `minute` | 否 | 当地钟表分钟，默认 `0` |
| `gender` | 是 | 同上 |
| `longitude` | 是 | 出生地经度，东经为正 |
| `timezone` | 否 | 出生地 IANA 时区，默认 `Asia/Shanghai` |
| `year_boundary` | 否 | `lunar_new_year`（默认）或 `lichun` |
| `language` | 否 | 同上 |

返回命宫与身宫、十二宫及各宫主星与亮度、辅星与煞星、五行局、生年四化、十年大限，以及所用的时间基准。

逐字段说明见 [`docs/reference.md`](docs/reference.md)（英文）。

## 示例

> 用 BaziChart 排一张八字命盘：女，1995 年 4 月 18 日上午 9 点 20 分，出生在成都。

AI 助手以 `longitude: 104.07`、`timezone: "Asia/Shanghai"`、`language: "zh-CN"` 调用 `cast_bazi`。返回节选：

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
  "source": { "name": "BaziChart.ai", "url": "https://bazichart.ai/zh-CN/chart/?…" }
}
```

不给出生地，服务器不排盘：

```json
{
  "ok": false,
  "error": { "code": "BIRTHPLACE_LONGITUDE_REQUIRED" },
  "hint": "Ask the user where they were born … Do not guess the birthplace or default to Beijing."
}
```

示例有删节。`hint` 是写给 AI 助手的提示。

## Agent 技能

技能告诉 AI 助手如何使用本服务：调用前要确认哪些信息、怎样传入出生地、怎样呈现结果、怎样处理报错。技能采用 [Agent Skills](https://agentskills.io) 格式。

```bash
npx skills add bazichart/bazichart-mcp
```

| 文件 | 用途 |
|---|---|
| [`skills/bazichart/SKILL.md`](skills/bazichart/SKILL.md) | 工作流程、呈现规则与报错处理 |
| [`references/conventions.md`](skills/bazichart/references/conventions.md) | 两处可选取法、时间校正、已知限制 |
| [`references/fields.md`](skills/bazichart/references/fields.md) | 参数与返回字段 |

技能文件为英文。使用前需先接入服务；Claude Code 用户通过[快速接入](#快速接入)里的插件可以一次装好。

## 已知限制

- **`Asia/Shanghai` 时区的紫微校时。** 出生地在该时区的，紫微盘只做经度校正，不扣夏令时、不计均时差。因此在中国 1986—1991 年夏令时期间，紫微与八字两个工具所用的时刻可能相差约一小时。其他时区按完整的真太阳时处理。
- **日界线附近四个地区。** 基里蒂马蒂、汤加、萨摩亚、查塔姆群岛出生的，排盘所用的太阳日期可能比当地日历早一天。
- **不返回叠宫。** 该字段为空，并附状态说明原因。
- **盘面术语为简体中文。** 天干地支、十神、星曜、宫位名称一律以简体中文返回，与 `language` 参数无关；需要其他语言时由 AI 助手在呈现时翻译。
- **需要明确的出生时间。** 只有日期或只有大概时段的，不排盘。
- **来源链接指向 BaziChart 排盘页**，不是刚算出的那张盘。
- **限额按 IP 地址计**：每分钟 30 次、每天 1000 次排盘。超出的调用返回 HTTP 429 及 `Retry-After`。

## 隐私

不使用账号或密钥，调用不与任何身份关联。限额按 IP 地址计数。应用日志只记录每次调用的工具名、状态和耗时，不记录出生资料。请求经由 Cloudflare 到达服务器。每次返回的来源链接带有 `utm_` 参数，用于标明访问来自 MCP 服务。网站的隐私政策见 [bazichart.ai/zh-CN/privacy](https://bazichart.ai/zh-CN/privacy/)。

## 署名

每次成功返回都带 `source.url` 和一句署名，AI 助手在展示命盘时应予引用。

## 支持

- 网站：[bazichart.ai](https://bazichart.ai)
- 面向普通用户的接入说明：[bazichart.ai/zh-CN/mcp](https://bazichart.ai/zh-CN/mcp/)
- 邮箱：support@bazichart.ai
- 问题与反馈：在本仓库提交 issue
- 安全问题上报：见 [SECURITY.md](SECURITY.md)

## 免责声明

命盘以及 AI 助手据此作出的任何解读，仅供娱乐消遣，不构成任何专业建议。

---

本仓库只包含托管服务的说明文档。排盘引擎为专有软件，不在此分发。本仓库的文档以 [CC BY 4.0](LICENSE) 许可发布。
