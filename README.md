<div align="center">

# Video Prompt Skill

**Convert movie / drama scripts into production-ready prompts for five AI video platforms — in one pass.**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Claude Skill](https://img.shields.io/badge/Claude-Skill-8a4fff)](https://docs.claude.com/en/docs/claude-code/skills)
[![Version](https://img.shields.io/badge/version-3.2-blue)](#whats-new-in-v32)
[![中文](https://img.shields.io/badge/lang-中文-red)](README.zh-CN.md)

</div>

One drama script row in → five platform-ready prompts out. Each prompt uses the right language (Chinese for Kling and Seedance, English for Veo / Grok / HappyHorse), the right reference syntax, the right multi-shot structure, and the right level of detail.

> **🆕 v3.2** introduces a strict shot format (`镜头N[start-end s]:`), six required dimensions per shot (景别 / 动作 / 微表情 / 光影 / 运镜 / 音效), a **50-entry micro-expression library**, and aligns Seedance output with the official Volcengine spec (Chinese 4-section structure). See [What's new in v3.2](#whats-new-in-v32).

---

## Table of contents

- [Why this skill](#why-this-skill)
- [What's new in v3.2](#whats-new-in-v32)
- [Supported platforms](#supported-platforms)
- [Quick start](#quick-start)
- [Input format](#input-format)
- [Output format](#output-format)
- [Platform selection](#platform-selection)
- [Strict shot format (v3.2)](#strict-shot-format-v32)
- [Six required dimensions](#six-required-dimensions)
- [Micro-expression library](#micro-expression-library)
- [Rhythm analysis](#rhythm-analysis)
- [Shot library](#shot-library)
- [Worked examples](#worked-examples)
- [Per-platform notes](#per-platform-notes)
- [Common pitfalls](#common-pitfalls)
- [FAQ](#faq)
- [Project structure](#project-structure)
- [Version history](#version-history)
- [Contributing](#contributing)
- [License](#license)

---

## Why this skill

Generating prompts for five video AI platforms by hand is tedious and error-prone:

- **Kling** needs Chinese prompts and Chinese `@元素名` references
- **Seedance** (v3.2) needs Chinese 4-section structure with `@图N + 名称` anchoring, time slices, and a 7-item anti-collapse tail
- **HappyHorse** needs pinyin element names and 2–4 multi-angle images per character
- **Grok** needs short English prompts with `@image1`
- **Veo** doesn't use textual markers at all — just an ordered image array

This skill takes one structured script row and emits all five (or any subset) in their native format, with consistent semantics across all of them. The trick: every row first becomes a Chinese director's storyboard (`sceneDesc` with `镜头N[start-end s]:` shots — six dimensions each), then translates into each platform's syntax.

**Key features:**

- ✅ All five platforms (Veo 4, Grok Imagine, Seedance 2.0, HappyHorse 1.0, Kling 3.0 Omni)
- ✅ Subset selection by JSON field OR natural language (`"只要 Kling"`, `"skip Grok"`)
- ✅ Semantic rhythm analysis — no fixed `N = round(time / 1.5)` formula
- ✅ **Strict v3.2 shot format** with self-check before every row
- ✅ **Six required dimensions** per shot — never silently drop one
- ✅ **50-entry micro-expression library** across 10 emotion categories
- ✅ 60+ camera shot library with emotion → shot-code mapping
- ✅ Physics keywords for Kling (`真实重力`, `重心转移`, `头发惯性`)
- ✅ Native audio handling for Veo / Seedance
- ✅ **Seedance 2.0 official Volcengine spec** compliance (Chinese 4-section + anti-collapse tail)
- ✅ Multi-shot orchestration (Kling `multi_prompt`, Seedance time slices)
- ✅ Streaming JSONL output for downstream pipelines

---

## What's new in v3.2

Four hard-constraint upgrades over v3.1:

### 1. Shot format hardening

Every shot in `sceneDesc` must use exactly `镜头N[start-end s]:` — Arabic numeral, half-width brackets, half-width `s`, half-width colon + space. The old `△[time]` form is now an error.

```diff
- △[0-2s] 中景，赵长安 缓慢盘膝坐下…
+ 镜头1[0-2s]: 中景，赵长安@图片2 缓慢盘膝坐下，眉心微皱呼吸放缓…
```

### 2. Six-dimension structured shots

Every shot description must explicitly include all six:

| # | Dimension | Required element |
|---|---|---|
| ① | 景别 (shot size) | 远景 / 全景 / 中景 / 近景 / 特写 / 极近特写 |
| ② | 主体动作 (action) | who + what + how (process) |
| ③ | 微表情 (micro-expression) | 眼神 / 眉眼 / 嘴角 / 呼吸 / 小动作 |
| ④ | 光影 (lighting) | direction + temperature + intensity + shadows |
| ⑤ | 运镜 (camera move) | push / pull / pan / tilt / track / rotate + speed |
| ⑥ | 音效 (audio) | ambient / action / emotional SFX |

If a dimension genuinely doesn't apply, write `静止 / 无运镜 / 无动作` explicitly — never silently omit.

### 3. 50-entry micro-expression library

A new on-demand reference file [`references/micro-expressions.md`](references/micro-expressions.md) ships 50 scene-tagged presets across 10 emotion categories. Each entry follows the formula:

> **eye landing + brow change + mouth change + breath pause + small action + applicable scene**

Use the library entries directly in the ③ dimension instead of inventing wording.

| Group | Entries | Use cases |
|---|---|---|
| 9.2.1 委屈/隐忍 | 6 | misunderstood / forced restraint / farewell |
| 9.2.2 紧张/慌乱/心虚 | 5 | exposed secret / hesitation / breakdown |
| 9.2.3 心动/重逢/情感 | 7 | first sight / reunion / object trigger |
| 9.2.4 冷淡/克制/距离感 | 4 | fake indifference / aristocratic distance |
| 9.2.5 怒意/反击/不服 | 4 | suppressed rage / cold counter |
| 9.2.6 权谋/试探/隐忍 | 11 | political tension / fake calm / killing intent flash |
| 9.2.7 崩溃/疯感/迷失 | 3 | obsession collapse / memory waking |
| 9.2.8 强者气场/神性/威压 | 4 | first love filter / sect master / divine sorrow |
| 9.2.9 强忍/坚守/信念 | 3 | bad news / misunderstanding cleared |
| 9.2.10 释然/结局/守护 | 3 | survival aftermath / silent guardian / ending peace |

### 4. Seedance 2.0 official Volcengine spec

Previously: English `[Image1]` + `Scene N / lens switch`. **Now**: Chinese **4-section** structure aligned with the official `byted-ark-seedance-pe v1.0` skill.

```
[① 全局基础设定]   @图1 角色1名称+描述，@图2 角色2，@图3 场景。
[② 时间片分镜]     0-X 秒：@图N 角色 在 @图M 场景 做什么，<6 dimensions>；
                  X-Y 秒：…；
[③ 音频/台词段]    台词：「…」；音效：…；环境音：…。
[④ 防崩坏画质尾]   4K 高清，细节丰富，<style>；面部稳定不变形、五官清晰、
                  人体结构正常、动作自然流畅、不僵硬、画面无卡顿、无闪烁。
```

Hard rules:
- **`@图N` must be followed by the name** — `@图1 赵长安`, never bare `@图1` (prevents Chinese tokenizer ambiguity)
- **One camera move per time slice** — split if a shot has push + pull
- **All seven anti-collapse items required** in the tail
- `references` shape is now `{tag, name, file}` (was `{type, file, index}`)

---

## Supported platforms

| Platform | Origin | Language | Reference syntax | Imgs per ref | Multi-shot |
|---|---|---|---|---|---|
| **Veo 4** | Google DeepMind | English | none (ordered array) | 1 | no |
| **Grok Imagine** | xAI | English | `@image1` | 1 | frame-chain |
| **Seedance 2.0** | ByteDance | **Chinese** | `@图N + 名称` | 1 | **time slices** `0-X 秒：…；` |
| **HappyHorse 1.0** | Alibaba | English | `@element_name` | **2–4 multi-angle** | shot array |
| **Kling 3.0 Omni** | Kuaishou | **Chinese** | `@中文名` | **4+ multi-angle** | `multi_prompt` |

**Strengths at a glance:**

- **Veo** — strongest prompt adherence, best for static / commercial / micro-expression
- **Grok** — fastest iteration, viral social-media aesthetic, in-image text rendering
- **Seedance** — best multi-shot in single generation, native audio, official spec compliance
- **HappyHorse** — best single-character portraits, native multilingual lip-sync
- **Kling** — best physics simulation, best action choreography, IP element reuse

---

## Quick start

### 1. Install

```bash
# user-level (available across all projects)
git clone https://github.com/liqi21cn/video-prompt-skill.git \
  ~/.claude/skills/video-prompt-skill

# OR project-level (only for current project)
git clone https://github.com/liqi21cn/video-prompt-skill.git \
  .claude/skills/video-prompt-skill
```

### 2. Verify

Restart Claude Code, then ask:

> 列出我可用的 skills

The skill should appear as `video-prompt`.

### 3. Use

```
帮我把这段剧本转成 Kling 和 Seedance 的提示词：
{ "tableData": [...], "assetLists": {...} }
```

The skill auto-triggers on phrases like *分镜 / 画面 prompts / video prompt / 即梦 / 可灵 / Veo / Grok*.

---

## Input format

Top-level structure:

```json
{
  "platforms": ["kling", "seedance"],     // optional, default all five
  "tableData":  [ /* script rows */ ],     // required
  "assetLists": { /* reference assets */ } // required
}
```

### `tableData[]` — script rows

| Field | Type | Required | Notes |
|---|---|---|---|
| `time` | number | yes | Duration in seconds. 2–15 typical. |
| `attribute` | string | yes | `人物·<名>` / `旁白` / `系统` / `环境` / `字幕` |
| `originalText` | string | yes | The dialogue / narration / system text |

### `assetLists` — reference assets

```json
{
  "assetLists": {
    "scenes":     [{ "name", "description", "multi_angle_urls": [...] }],
    "characters": [{ "name", "description", "multi_angle_urls": [...] }],
    "items":      [{ "name", "description", "multi_angle_urls": [...] }]
  }
}
```

**`multi_angle_urls` sizing:**

| Targeted platforms | Recommended URL count |
|---|---|
| Only Veo / Grok | 1 frontal |
| Only Seedance | 1–3 (each angle becomes its own `@图N`) |
| Includes HappyHorse | **2–4 multi-angle** |
| Includes Kling | **4 multi-angle** (front / side / 3/4 / back) |

Full schema and validation rules: [`references/input-schema.md`](references/input-schema.md).

---

## Output format

Streaming **JSONL** — one JSON object per row.

```json
{
  "time": 4,
  "attribute": "人物·李雷",
  "originalText": "他长叹一声，转身离去。",
  "sceneDesc": "镜头1[0-2s]: 近景，李雷@图片2 ... 镜头2[2-4s]: 中景背影 ...",
  "imageRefs": ["李雷"],

  "veoPrompt":        { "prompt": "...", "reference_images": [...], "negative_prompt": "..." },
  "grokPrompt":       { "prompt": "...", "image_urls": [...], "duration": 4 },
  "seedancePrompt":   { "prompt": "<4-section Chinese>", "references": [{"tag": "@图1", "name": "...", "file": "..."}] },
  "happyhorsePrompt": { "prompt": "...", "elements": [...] },
  "klingPrompt":      { "prompt": "...", "elements": [...], "multi_prompt": [...] },

  "prompt": ""
}
```

**Rules:**

- Fields `time`, `attribute`, `originalText`, `sceneDesc`, `imageRefs`, `prompt: ""` always appear
- Platform fields appear **only if requested** — non-selected platforms are *completely omitted*
- `sceneDesc` uses the **v3.2 strict format** `镜头N[start-end s]:` (no more `△[time]`)
- Output is one row per line — no array brackets, no markdown fences, no commentary

---

## Platform selection

By default, all five platforms are generated. You can narrow this two ways:

### Method 1 — Natural language

| You say | Resolves to |
|---|---|
| `只生成 Kling 的提示词` | `["kling"]` |
| `只要 Veo 和 Seedance` | `["veo", "seedance"]` |
| `just Veo` | `["veo"]` |
| `skip Grok and Veo` | `["seedance", "happyhorse", "kling"]` |
| `exclude HappyHorse` | `["veo", "grok", "seedance", "kling"]` |

### Method 2 — JSON field

```json
{ "platforms": ["kling", "seedance"], "tableData": [...], "assetLists": {...} }
```

Aliases (case-insensitive):

| Canonical | Recognized aliases |
|---|---|
| `veo` | Veo, Veo 4, Google Veo |
| `grok` | Grok, Grok Imagine, xAI Grok |
| `seedance` | Seedance, Seedance 2.0, 即梦 |
| `happyhorse` | HappyHorse, Happy Horse, 快马 |
| `kling` | Kling, Kling 3.0 Omni, 可灵 |

**If both are present**, the JSON field wins.

---

## Strict shot format (v3.2)

Every shot in `sceneDesc` **must** follow:

```
镜头N[起始s-结束s]: <six-dimension description>
```

### Correct ✓

```
镜头1[0-1.5s]: 中景，赵长安@图片2 缓慢盘膝坐下，眉心微皱呼吸放缓，烛火暖光从右侧打亮面部，缓慢推入聚焦面部，环境音衣料摩擦声
```

### Wrong ✗

| Wrong form | Why |
|---|---|
| `△[0-2s] xxxxx` | Missing `镜头N` + colon |
| `0-2秒：xxxxx` | Missing brackets + missing `镜头N` + Chinese 秒 |
| `(0-1.5s) xxxxx` | Round parens instead of square brackets |
| `第1段 0-1.5s xxxxx` | Used `第N段` instead of `镜头N` |
| `镜头1：0-1.5s xxxxx` | Time not in brackets |
| `镜头1[0,1.5s]: xxxxx` | Comma instead of hyphen |
| `Shot1[0-1.5s]: xxxxx` | Used English `Shot` |
| `镜头一[0-1.5s]: xxxxx` | Chinese numeral instead of Arabic |

### Pre-output self-check (every row)

```
□ Does every shot start with 镜头N[start-end s]: ?
□ Are all six dimensions explicitly present in each shot?
□ Are shot numbers consecutive from 1?
□ Does Σ shot durations equal the `time` field?
□ Is each shot ≥ 0.8s and ≤ 5s?
□ Are shots time-continuous (no gaps, no overlaps)?
□ Are micro-expressions physicalized (not abstract emotion words)?
```

---

## Six required dimensions

Every shot description must explicitly contain all six. Order can shift, but presence cannot.

```
镜头N[start-end s]:
  ① 景别 (shot size)         — 中景
  ② 主体动作 (action)         — 赵长安@图片2 缓慢盘膝坐下
  ③ 微表情 (micro-expression) — 眉心微皱呼吸放缓
  ④ 光影 (lighting)           — 烛火暖光从右侧打亮面部
  ⑤ 运镜 (camera move)        — 缓慢推入聚焦面部
  ⑥ 音效 (audio)              — 环境音衣料摩擦声
```

**Sparse dimensions:** If a frozen still has no camera move, write `静止 / 无运镜` explicitly. If a shot has no subject action (pure environment), write `无主体动作`. Never silently drop a dimension.

---

## Micro-expression library

50 scene-tagged presets in [`references/micro-expressions.md`](references/micro-expressions.md). Each entry:

> **eye landing + brow change + mouth change + breath pause + small action + applicable scene**

### How to use

1. **Match emotion → category** (e.g. 委屈 → 9.2.1)
2. **Match scene → entry** (e.g. "被误会" → #01 强忍委屈)
3. **Copy the physical description verbatim** into the ③ dimension. Don't include `#NN` or the scene tag.
4. **Trim** for short shots — eye landing + one small action often suffices
5. **Stack** for complex emotion (e.g. #18 悔意涌上 + #01 强忍委屈)

### Sample entries

| # | Tag | Description | Use case |
|---|---|---|---|
| 01 | 强忍委屈 | 眼尾泛红但泪不落，眉心轻拧又松开，嘴角向下压住，下巴轻颤，视线从对方脸上滑开像把解释咽回去 | 被误会、背锅、临别强撑 |
| 19 | 破防瞬间 | 原本平静的眼神忽然碎掉，瞳孔轻颤，嘴角努力维持却失败，鼻翼发酸，眼泪在眶里打转但还没掉 | 听到最痛的一句话 |
| 46 | 看到旧物 | 目光落在物件上不动，眉眼迅速柔软又泛酸，呼吸一滞，指尖轻颤想伸出又收回 | 看到亡者遗物、旧信物 |

For English platforms (Veo / Grok / HappyHorse), translate the physical observations directly — they translate cleanly. Scene tags (e.g. "邪魅试探") don't.

---

## Rhythm analysis

Shot count is **not** derived from time. Four dimensions decide:

| Dimension | What to look for | Effect |
|---|---|---|
| **Action density** | Explosive verbs (炸/劈/扑/碎) vs static descriptions | Extreme → 0.8–1.5s dense cuts; Low → 3–5s long takes |
| **Emotional arc** | Calm / tension / climax / turning point | Maps to rhythm curve (slow / build / fast / freeze) |
| **Dialogue presence** | Has spoken lines? | Each line gets ≥ 2s dwell time |
| **Information density** | Distinct visual points in the sentence | Each major point gets at least one shot |

### Hard constraints (always enforced)

- Each shot **≥ 0.8s** (below this is imperceptible)
- Each shot **≤ 5s** (8s for special long takes)
- ∑ shot durations **= `time` exactly** — no drift
- No gaps between shots
- Strict v3.2 format with all six dimensions

### Example: same duration, different cuts

```
6 seconds, "我想……我们或许该回去了。"
  → 1–2 long shots:  镜头1[0-3s] speaking shot  镜头2[3-6s] reaction

6 seconds, "他猛地侧身闪开，反手一刀劈下，敌人胸口炸开血花。"
  → 5–6 dense shots:  dodge | regrip | swing | contact | blood mist | aftermath
```

Full decision rules: [`references/rhythm-rules.md`](references/rhythm-rules.md).

---

## Shot library

60+ camera shots in 9 categories, each with a code:

| Cat. | Theme | Example codes |
|---|---|---|
| **A** | Push / pull movements | `A01` 缓慢推入, `A03` 缓慢拉出, `A08` 希区柯克变焦 |
| **B** | Trajectories | `B04` 弧形绕摄, `B05` 环绕运镜 |
| **C** | Speed / special | `C01` 手持灵动, `C06` 急停定帧 |
| **D** | Angle / composition | `D01` 荷兰角, `D05` 无力仰角 |
| **E** | Detail / emotion close-ups | `E01` 眼神特写, `E09` 眼泪滑落 |
| **F** | Psychology / consciousness | `F01` 旋转晕眩, `F02` 模糊转清 |
| **G** | Action / combat | `G07` 子弹时间, `G08` 破门而入 |
| **H** | Transitions | `H04` 回忆叠化, `H06` 长镜压抑 |
| **I** | Effects / styles | `I06` 水墨晕染, `I11` 魔法释放 |

**Selection algorithm:** Emotion → Action → Style cascade. Emotion wins on conflict.

Full library + emotion lookup: [`references/shot-library.md`](references/shot-library.md).

---

## Worked examples

### Example A — Full five platforms (v3.2 format)

**Input:**

```json
{
  "tableData": [
    { "time": 4, "attribute": "人物·李雷",
      "originalText": "他看到她递过来的旧信物，眼眶突然就湿了。" }
  ],
  "assetLists": {
    "characters": [
      { "name": "李雷", "description": "黑发青年，现代休闲装",
        "multi_angle_urls": ["lilei_1.jpg", "lilei_2.jpg"] }
    ],
    "items": [
      { "name": "旧玉佩", "description": "翠绿色雕花边缘有磨损",
        "multi_angle_urls": ["jade_1.jpg"] }
    ]
  }
}
```

**Rhythm analysis:** Low action density + emotional turn + 2 visual points → 2 long shots.

**Micro-expression library application:** #46 看到旧物 + #07 失而复得 (stacked).

**Output (abbreviated, showing v3.2 strict shot format):**

```json
{
  "time": 4,
  "sceneDesc": "镜头1[0-1.5s]: 近景，李雷@图片2 站在庭院夜景@图片1 里看着对方递过来的旧玉佩@图片3，目光落在物件上不动眉眼迅速柔软又泛酸，灯笼昏黄暖光从右侧打亮面部左半边在阴影里，镜头微微跟随玉佩落下的轨迹缓慢推入，落叶沙沙声与远处隐约的虫鸣 镜头2[1.5-4s]: 特写，李雷指腹轻轻摩挲玉佩边缘嘴角像要笑眼眶却先湿，眼眶瞬间湿润瞳孔放大嘴角想笑却先发抖鼻尖微红手伸到半空停住像怕一碰就消失，灯笼暖光打在玉佩上反射出温润绿光，固定长镜头不动，环境音放大轻微呼吸声与心跳般的鼓点",
  "imageRefs": ["李雷", "旧玉佩"],
  "...": "five platform prompts here"
}
```

**Expanded `seedancePrompt` (v3.2 official 4-section Chinese format):**

```json
{
  "seedancePrompt": {
    "prompt": "@图1 李雷现代休闲装年轻男性，@图2 旧玉佩翠绿色雕花边缘有磨损，@图3 庭院夜景灯笼昏黄落叶满地。\n\n0-1.5 秒：@图1 李雷站在@图3 庭院夜景里看着对方递过来的@图2 旧玉佩，目光落在物件上不动眉眼迅速柔软又泛酸，灯笼昏黄暖光从右侧打亮面部左半边在阴影里，镜头微微跟随玉佩落下的轨迹缓慢推入；\n1.5-4 秒：@图1 李雷指腹轻轻摩挲@图2 旧玉佩边缘嘴角像要笑眼眶却先湿，眼眶瞬间湿润瞳孔放大嘴角想笑却先发抖鼻尖微红手伸到半空停住像怕一碰就消失，灯笼暖光打在玉佩上反射出温润绿光泪光与玉佩形成呼应，镜头固定长镜头不动让情绪自然流出。\n\n环境音：落叶沙沙声、远处隐约的虫鸣、轻微呼吸声与心跳般的低沉鼓点。\n\n4K 高清，细节丰富，都市情感剧风格，暖光低饱和柔焦质感，写实电影情感氛围，整体氛围温柔治愈带着克制的悲伤；面部稳定不变形、五官清晰、人体结构正常、动作自然流畅、不僵硬、画面无卡顿、无闪烁。",
    "references": [
      {"tag": "@图1", "name": "李雷", "file": "lilei_1.jpg"},
      {"tag": "@图2", "name": "旧玉佩", "file": "jade_1.jpg"},
      {"tag": "@图3", "name": "庭院夜景", "file": "scene_yard.jpg"}
    ]
  }
}
```

Notice in the Seedance prompt:
- **Four sections** separated by `\n\n`: 全局段 / 时间片段 / 环境音段 / 防崩坏画质段
- Every `@图N` anchored with name (`@图1 李雷`, never bare `@图1`)
- Chinese time slices (`0-1.5 秒：` with full-width 秒 + space)
- Quality tail ends with all seven anti-collapse items
- One camera move per slice (`缓慢推入` / `固定长镜头不动`)

### Example B — Subset (only Kling + Seedance)

**Request:** *"只生成 Kling 和 Seedance 的"*

**Output:** the same row, but `veoPrompt`, `grokPrompt`, `happyhorsePrompt` are **completely absent** — not `null`, not `{"skip": true}`, just omitted.

---

## Per-platform notes

### Veo 4

- Don't write precise timings (`"walks for 3 seconds then turns"` will fail)
- Don't write multiple story arcs in one prompt
- **Always include `Audio:` segment** — wastes the native audio otherwise
- 3-image reference cap is a real limit

### Grok Imagine

- Hard **10-second cap** (some implementations 15s)
- Keep prompts **under 100 characters** for best results
- Skip generic adjectives like "4K", "high quality" — they're noise
- For sequences > 10s, use frame-chain (last frame → next input)

### Seedance 2.0 (heavily changed in v3.2)

- **Chinese only** (per official Volcengine spec) — the only exception is when the original dialogue is itself English
- **4-section structure required**: 全局 → 时间片 → 音频 → 防崩坏尾
- **`@图N` must be followed by name**: `@图1 赵长安`, never bare `@图1`
- **One camera move per time slice** — split if there's push + pull
- **Quality tail mandatory** with all seven anti-collapse items:
  > `面部稳定不变形、五官清晰、人体结构正常、动作自然流畅、不僵硬、画面无卡顿、无闪烁`
- `references` shape: `{"tag": "@图1", "name": "赵长安", "file": "url"}`
- Up to 9 images + 3 videos + 3 audio refs
- Background drift in long takes is a known limitation
- Deprecated forms (will not work): `[Image1]`, `Scene N`, `lens switch`, `[asset-xxx]`

### HappyHorse 1.0

- **Element name must be pinyin or English** — Chinese in `@` refs can fail
- Multi-angle (2–4 urls per element) is its superpower
- Description field heavily affects identity — write it carefully
- Multi-shot capped at 5 shots

### Kling 3.0 Omni

- **Write prompts in Chinese** for best quality
- Physics keywords meaningfully improve realism:
  - `真实重力` — gravity-correct motion
  - `重心转移` — weight shift in steps
  - `脚跟先落地` — heel-first natural walking
  - `布料下垂` — fabric drape
  - `头发惯性` — hair momentum in turns
  - `液体粘度` — fluid viscosity
- Dialogue trigger: `@角色名 直视镜头，缓缓开口说道："..."`
- For 4+ shots, also produce `multi_prompt` (up to 6 shots)

Full per-platform translation guide: [`references/platform-adaptations.md`](references/platform-adaptations.md).

---

## Common pitfalls

1. **Reverting to v3.1 `△[time]` format** — v3.2 strict format is non-negotiable
2. **Silently omitting a dimension** — all six required; write `静止/无运镜/无动作` explicitly if absent
3. **Inventing micro-expressions** when §9.2 library covers it — browse the library first
4. **Forgetting `Audio:` in Veo / 音频段 in Seedance** — both have native audio
5. **Using English in Seedance** — v3.2 mandates Chinese per official spec
6. **Using `[Image1]` / `Scene N / lens switch` in Seedance** — deprecated as of v3.2
7. **`@图N` without a following name in Seedance** — always anchor: `@图1 赵长安`
8. **Multiple camera moves in one Seedance slice** — split into two slices
9. **Missing 4K + anti-collapse tail in Seedance** — all seven items mandatory
10. **Using Chinese in HappyHorse element names** — stick to pinyin or English
11. **Forgetting `multi_prompt` for Kling action sequences** with 4+ shots
12. **Mismatched time totals** — verify Σ(shot durations) === `time` before output
13. **Repeating `@图片1` for scenes** — scene marked only in the first shot of the row
14. **Including scene names in `imageRefs`** — only characters and items
15. **Abstract emotion words** — never `"悲伤"` or `"happy"`; physicalize
16. **Writing `null` for unselected platforms** — just omit the field entirely
17. **Mis-resolving aliases** — `可灵` → `kling`, `即梦` → `seedance`, `快马` → `happyhorse`

---

## FAQ

**Q: My input is plain narrative text, not structured JSON. Can the skill still help?**
A: Yes — the skill will first help you format raw text into the `tableData` schema, then proceed.

**Q: What if my character isn't in `assetLists`?**
A: The character is described by name only (no `@` reference). All platform prompts fall back to text description. The skill flags `"_no_ref": ["X"]`.

**Q: How is `time` enforced?**
A: The skill verifies Σ(shot durations) === `time` exactly. Each shot is ≥ 0.8s and ≤ 5s. If the requested duration can't fit the rhythm, the skill warns.

**Q: Can I use this without Claude?**
A: The skill is built for Claude (via `SKILL.md` description triggering). The reference docs are usable as a manual prompt-engineering guide too, but the auto-trigger flow needs Claude.

**Q: Does the skill fetch reference image URLs?**
A: No. URL reachability is downstream — the skill just passes URLs through. Use any image host.

**Q: How do I update to a new version?**
A: `cd ~/.claude/skills/video-prompt-skill && git pull`.

**Q: Can I add my own platform?**
A: Fork, add a new section to `references/platform-adaptations.md`, add the ID to the canonical list in `SKILL.md`, update the output schema. PR welcome.

**Q: I have a v3.1 pipeline that expects `△[time]` format and `[Image1]` Seedance. How do I migrate?**
A: v3.2 is a breaking format change. Update your downstream parsers to expect `镜头N[start-end s]:` in `sceneDesc` and the 4-section Chinese structure in `seedancePrompt`. The output schema field names stay the same — only the *content* of `sceneDesc` and `seedancePrompt.prompt` changed.

---

## Project structure

```
video-prompt-skill/
├── SKILL.md                          # Skill entry — schema, workflow, pitfalls
├── README.md                         # This file (English)
├── README.zh-CN.md                   # 中文版
├── LICENSE                           # MIT
└── references/
    ├── input-schema.md               # Input format + validation
    ├── rhythm-rules.md               # 4-dimension rhythm analysis
    ├── shot-library.md               # 60+ camera shots (A01–I12) + emotion lookup
    ├── platform-adaptations.md       # Per-platform translation guide (v3.2 Seedance)
    ├── micro-expressions.md          # 🆕 v3.2: 50 micro-expression presets
    └── full-prompt-v3.2.md           # Full v3.2 system prompt archive
```

References are loaded **on demand** by Claude — not all upfront. This keeps context lean.

---

## Version history

| Area | v3.0 | v3.1 | **v3.2** |
|---|---|---|---|
| Shot count | `N = round(time / 1.5)` | Semantic 4-dimension analysis | Same as v3.1 |
| Platform selection | Always all five | NL or JSON subset | Same as v3.1 |
| Skipped platform output | (n/a) | Completely omitted | Same as v3.1 |
| Shot format | `△[time]` | `△[time]` | **`镜头N[start-end s]:` strict** |
| Per-shot dimensions | Free-form | Free-form | **6 required: 景别 / 动作 / 微表情 / 光影 / 运镜 / 音效** |
| Micro-expressions | Ad-hoc | Ad-hoc | **50-entry library across 10 categories** |
| Seedance format | English `[Image1]` + `Scene N / lens switch` | Same as v3.0 | **Chinese 4-section + `@图N + 名称` + anti-collapse tail** |
| Pre-output self-check | Implicit | Implicit | **8-item explicit checklist per row** |
| Shot library | 40 shots | 60+ shots, A01–I12 | Same as v3.1 |
| Physics keywords | (n/a) | Kling-specific list | Same as v3.1 |

---

## Contributing

Issues and PRs welcome.

Good first PR ideas:
- Add a new shot to `references/shot-library.md` (with code + emotion mapping)
- Add a new micro-expression to `references/micro-expressions.md` (with scene tag + 5-element description)
- Document a new platform behavior in `references/platform-adaptations.md`
- Add a worked example for an edge case
- Translate `references/*.md` to another language

For substantial changes (new platform support, rhythm algorithm changes, format hardening), open an issue first.

---

## License

[MIT](LICENSE) — use freely in commercial and personal projects.

---

<div align="center">

**Found this useful?** ⭐ Star the repo so others can find it.

</div>
