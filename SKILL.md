---
name: video-prompt
description: Convert a movie/drama script (tableData with time, attribute, originalText fields + asset lists with reference images) into production-ready prompts for AI video generation platforms - Veo 4, Grok Imagine, Seedance 2.0, HappyHorse 1.0, and Kling 3.0 Omni. Users can request all five platforms (default) or specify a subset (e.g. "只生成 Kling 的", "just Veo and Seedance"). Use this skill whenever the user provides script data and asks for video generation prompts, storyboards, AI video prompts, multi-shot breakdowns, 分镜, 画面 prompts, or asks to convert text/scripts into video AI prompts. Also use when the user mentions any of these specific platforms by name and wants script-to-prompt conversion, or when they need multi-platform video prompt generation. v3.2 enforces a strict shot format (镜头N[start-end s]:) and six required dimensions per shot (景别/动作/微表情/光影/运镜/音效), and ships a 50-entry micro-expression library for precise emotional performance. Seedance 2.0 output now follows the official Volcengine spec (Chinese, four-section structure with @图N+name anchoring and time slices, anti-collapse tail). Handles rhythm-based shot pacing (not fixed formulas), platform-specific reference image syntax adaptation, and emotion-to-lighting mapping.
---

# Video Prompt v3.2

A skill for converting drama/movie scripts into multi-platform AI video generation prompts. Outputs can be adapted for any subset of **Veo 4, Grok Imagine, Seedance 2.0, HappyHorse 1.0, and Kling 3.0 Omni** — each with its own reference syntax and language preference.

## What's new in v3.2

Four hard-constraint upgrades over v3.1:

1. **Shot format hardening** — Every shot in `sceneDesc` must use exactly `镜头N[start-end s]: ...` (Arabic numeral, half-width brackets, half-width `s`, half-width colon + space). The old `△[time]` form is now an error. See **§4 Shot format hard constraint** in `references/full-prompt-v3.2.md`.
2. **Six-dimension structured shots** — Every shot description must explicitly contain: **景别 (shot size) + 主体动作 (action) + 微表情 (micro-expression) + 光影 (lighting) + 运镜 (camera move) + 音效 (audio)**. No dimension can be silently omitted.
3. **50-entry micro-expression library** — `references/full-prompt-v3.2.md` §9.2 ships 50 scene-tagged micro-expression presets across 10 emotion categories (委屈/紧张/心动/冷淡/怒意/权谋/崩溃/强者气场/坚守/释然). For the "③ micro-expression" dimension, **prefer pulling from this library** over inventing wording from scratch.
4. **Seedance 2.0 output aligned with official Volcengine spec** — Previously English `[Image1]` + `Scene N / lens switch`; now **Chinese, four-section structure**: ① global setup with `@图N + name`, ② time slices (`0-X 秒：...；`), ③ audio/dialogue segment, ④ quality+anti-collapse tail (`4K 高清，细节丰富...面部稳定不变形、五官清晰...无闪烁`). One camera move per slice. The `references` array now uses `{tag, name, file}` shape. See §6.4 of `references/full-prompt-v3.2.md`.

## When to use this skill

Use this skill when the user provides:
- Script data in JSON form (with `tableData` containing `time`, `attribute`, `originalText` fields)
- Asset lists with character/scene/item names and reference images
- Any request to generate video prompts, storyboards, shot breakdowns, or "分镜"

Also use when the user mentions converting scripts/dialogue/narration to AI video, or names any of the five supported platforms (Veo, Grok, Seedance, HappyHorse, Kling).

If the user provides only narrative text without proper script structure, first help them format it into the expected `tableData` schema (see `references/input-schema.md`), then proceed.

## Platform selection

By default, generate prompts for all five platforms. But the user can request a subset in two ways:

### Method 1: Natural language in the request

Examples of phrases that narrow the platform selection:
- "只生成 Kling 的提示词" → `["kling"]`
- "只要 Veo 和 Seedance" → `["veo", "seedance"]`
- "just Veo" → `["veo"]`
- "I only need Kling and HappyHorse" → `["kling", "happyhorse"]`
- "skip Grok and Veo" → `["seedance", "happyhorse", "kling"]`
- "exclude HappyHorse" → `["veo", "grok", "seedance", "kling"]`

### Method 2: Structured `platforms` field in input JSON

```json
{
  "platforms": ["kling", "seedance"],
  "tableData": [...],
  "assetLists": {...}
}
```

### Canonical platform IDs (case-insensitive)

| ID | Aliases the user might use |
|---|---|
| `veo` | Veo, Veo 4, Veo4, Google Veo |
| `grok` | Grok, Grok Imagine, GrokImagine, xAI Grok |
| `seedance` | Seedance, Seedance 2.0, 即梦, ByteDance Seedance |
| `happyhorse` | HappyHorse, Happy Horse, HappyHorse 1.0, 快马 |
| `kling` | Kling, Kling 3.0 Omni, 可灵, 可灵 Omni, KuaiShou Kling |

### Resolution priority

If both the natural-language request AND the JSON `platforms` field are present, **the JSON field wins**. Mention this briefly in your response so the user knows.

If neither is specified, default to all five: `["veo", "grok", "seedance", "happyhorse", "kling"]`.

### Output rule when subset is selected

Only the requested platform fields appear in the output JSON. Non-requested platforms are **omitted entirely** (do not write `null` or `{"skip": true}` — just leave them out).

## Format enforcement (v3.2 hard constraint)

Every shot in `sceneDesc` **must** follow this exact pattern. Anything else is an error and must be self-corrected before output.

### The only legal format

```
镜头N[起始s-结束s]: <six-dimension description>
```

### Examples

✓ **Correct**:
```
镜头1[0-1.5s]: 中景，赵长安@图片2 缓慢盘膝坐下，眉心微皱呼吸放缓，烛火暖光从右侧打亮面部，缓慢推入聚焦面部，环境音衣料摩擦声
```

✗ **Wrong** (any of these is an error):

| Wrong form | Why it's wrong |
|---|---|
| `△[0-2s] xxxxx` | Missing `镜头N` + colon |
| `0-2秒：xxxxx` | Missing brackets + missing `镜头N` + Chinese 秒 |
| `(0-1.5s) xxxxx` | Round parens instead of square brackets |
| `第1段 0-1.5s xxxxx` | Used `第N段` instead of `镜头N` |
| `镜头1：0-1.5s xxxxx` | Time not in brackets |
| `镜头1[0,1.5s]: xxxxx` | Comma instead of hyphen |
| `Shot1[0-1.5s]: xxxxx` | Used English `Shot` |
| `镜头一[0-1.5s]: xxxxx` | Chinese numeral instead of Arabic |

### Six required dimensions per shot

| # | Dimension | Required element | Example |
|---|---|---|---|
| ① | 景别 (shot size) | 远景/全景/中景/近景/特写/极近特写 | 中景 |
| ② | 主体动作 (action) | who + what + how (process) | 赵长安@图片2 缓慢盘膝坐下 |
| ③ | 微表情 (micro-expression) | 眼神/眉眼/嘴角/呼吸/小动作 (see §9.2 library) | 眉心微皱呼吸放缓 |
| ④ | 光影 (lighting) | source direction + temperature + intensity + shadows | 烛火暖光从右侧打亮面部 |
| ⑤ | 运镜 (camera move) | push/pull/pan/tilt/track/rotate + speed | 缓慢推入聚焦面部 |
| ⑥ | 音效 (audio) | ambient / action / emotional SFX | 环境音衣料摩擦声 |

**Writing template** (order can shift, but all six must appear):
```
镜头N[start-end s]: ①shot-size, ②subject+action, ③micro-expression, ④lighting, ⑤camera, ⑥audio
```

If a dimension genuinely doesn't apply (e.g. a frozen still has no camera move), explicitly write `静止 / 无运镜 / 无动作` — never silently drop it.

### Pre-output self-check (run on every row)

```
□ Does every shot start with 镜头N[start-end s]: ?
□ Are all six dimensions explicitly present in each shot?
□ Are shot numbers consecutive from 1?
□ Does the sum of shot durations equal the `time` field?
□ Is each shot ≥ 0.8s and ≤ 5s?
□ Are shots time-continuous (no gaps, no overlaps)?
□ Are micro-expressions physicalized (not abstract emotion words)?
```

If any item fails, rewrite before emitting the row.

## Micro-expression library (v3.2)

50 scene-tagged presets in `references/full-prompt-v3.2.md` §9.2 (also available as `references/micro-expressions.md` for on-demand loading). Each entry follows the same five-element formula:

> **eye landing + brow change + mouth change + breath pause + small action + applicable scene**

### Categories (10 groups)

| Group | Entries | Use cases |
|---|---|---|
| 9.2.1 委屈/隐忍 | 6 (#01, 14, 18, 32, 37, 41) | misunderstood / forced restraint / farewell |
| 9.2.2 紧张/慌乱/心虚 | 5 (#03, 08, 11, 19, 28) | exposed secret / hesitation / breakdown |
| 9.2.3 心动/重逢/情感 | 7 (#04, 07, 17, 36, 44, 45, 46) | first sight / reunion / object trigger |
| 9.2.4 冷淡/克制/距离感 | 4 (#05, 22, 33, 49) | fake indifference / aristocratic distance |
| 9.2.5 怒意/反击/不服 | 4 (#06, 09, 10, 47) | suppressed rage / cold counter / battle scar smile |
| 9.2.6 权谋/试探/隐忍 | 11 (#02, 13, 15, 20-21, 23-27, 30) | political tension / fake calm / killing intent flash |
| 9.2.7 崩溃/疯感/迷失 | 3 (#29, 35, 48) | obsession collapse / memory waking |
| 9.2.8 强者气场/神性/威压 | 4 (#31, 34, 38, 39) | first love filter / sect master / divine sorrow |
| 9.2.9 强忍/坚守/信念 | 3 (#12, 16, 40) | bad news / misunderstanding cleared |
| 9.2.10 释然/结局/守护 | 3 (#42, 43, 50) | survival aftermath / silent guardian / ending peace |

### How to use

1. **Pick by scene tag** — Map the `originalText` emotional context to the closest category, then to the closest entry.
2. **Copy the physical description verbatim into the ③ dimension** of the shot — don't include the # number or scene tag.
3. **Trim if needed** — 50 entries average ~50 chars each. For a short shot, take just the most distinctive 1-2 details (eye landing + one small action usually suffices).
4. **Combine across entries** — Complex emotion can stack two (e.g. "悔意涌上 + 强忍委屈"). See `references/full-prompt-v3.2.md` §12.2 for a worked stacked example.
5. **Don't translate** — These are written in Chinese. For Veo / Grok / HappyHorse (English), translate each entry's physical observations rather than the scene tag. The physical details (e.g. "pupils contract, lips press white") translate cleanly; the scene-tag names (e.g. "邪魅试探") don't.

## Core workflow

**Step 0**: Determine the selected platforms (see "Platform selection"). Default = all five.

For each row in `tableData`:

1. **Read `originalText` and analyze rhythm** — Decide shot count from action density, emotional arc, dialogue presence, information density. No fixed formula. See `references/rhythm-rules.md`.

2. **Assign per-shot durations** — Each shot ≥ 0.8s, ≤ 5s; sum equals `time`. Durations can be uneven.

3. **Build `sceneDesc` in strict v3.2 format** — `镜头N[start-end s]:` prefix, six dimensions per shot. The director-readable canonical mid-form that all platform prompts derive from.

4. **Pull micro-expressions from §9.2 library when applicable** — Prefer library entries over inventing wording, especially for emotion-heavy rows.

5. **Apply shot selection algorithm** — Match emotion → action → style from the shot library. See `references/shot-library.md`.

6. **Translate to platform prompts** — Convert `@图片N` markers into each requested platform's specific reference syntax. Skip platforms not in selected list. See `references/platform-adaptations.md`.

7. **Run the self-check (8 steps above) before emitting the row.**

8. **Output JSONL** — One JSON object per row, streamed row-by-row. Only include requested platform fields.

## Output schema (strict)

```json
{
  "time": <number>,                                              // always
  "attribute": "<unchanged from input>",                         // always
  "originalText": "<unchanged from input>",                      // always
  "sceneDesc": "<镜头1[0-2s]: desc 镜头2[2-4s]: desc ...>",      // always, v3.2 format
  "imageRefs": ["<names>"],                                      // always

  "veoPrompt": {                                                 // conditional
    "prompt": "<English, 5-element structure>",
    "reference_images": ["<urls, max 3, identity-clearest first>"],
    "negative_prompt": "<exclusions>"
  },
  "grokPrompt": {                                                // conditional
    "prompt": "<English, short, with @image1 refs>",
    "image_urls": ["<urls, max 3>"],
    "duration": <number>
  },
  "seedancePrompt": {                                            // conditional
    "prompt": "<Chinese, 4-section structure: global / time-slices / audio / quality-tail>",
    "references": [{"tag": "@图1", "name": "<role or scene name>", "file": "<url>"}]
  },
  "happyhorsePrompt": {                                          // conditional
    "prompt": "<English, simple, with @element_name refs>",
    "elements": [{
      "name": "<pinyin or english>",
      "description": "<english>",
      "element_input_urls": ["<2-4 multi-angle urls>"]
    }]
  },
  "klingPrompt": {                                               // conditional
    "prompt": "<Chinese, with @中文名 refs, physics keywords>",
    "elements": [{
      "name": "<chinese>",
      "description": "<chinese>",
      "element_input_urls": ["<multi-angle urls>"]
    }],
    "multi_prompt": [{"prompt": "<...>", "duration": <number>}]
  },

  "prompt": ""                                                   // always (backward compat)
}
```

## Critical platform differences (memorize)

| Platform | Language | Reference syntax | Imgs per ref | Multi-shot |
|---|---|---|---|---|
| Veo 4 | English | none (ordered array) | 1 | no |
| Grok Imagine | English | `@image1` | 1 | no (frame chain) |
| Seedance 2.0 | **Chinese** | `@图N + 名称` (Chinese) | 1 | **time slices `0-X 秒：...；`** |
| HappyHorse 1.0 | English | `@element_name` | **2-4 multi-angle** | shot array |
| Kling 3.0 Omni | Chinese | `@中文名` | **4+ multi-angle** | `multi_prompt` |

**Language reason:** Kling and **Seedance** are Chinese-native (字节系/可灵 trained primarily on Chinese corpora); Veo / Grok / HappyHorse are English-native. The official Volcengine prompt-engineering skill (byted-ark-seedance-pe v1.0) prescribes Chinese as the canonical Seedance language.

**Element systems:** HappyHorse and Kling support named elements bundling multiple angles. Use ALL `multi_angle_urls` for these two. For Veo / Grok, use only the first (frontal) URL per entity. **Seedance**: each angle becomes a separate `@图N + 名称` reference (so 3 character angles → `@图1`, `@图2`, `@图3` all pointing at the same character).

## Seedance 2.0 official format (v3.2 hard spec)

Aligned with the official Volcengine prompt-engineering skill (`byted-ark-seedance-pe v1.0`). For any row generating `seedancePrompt`, the output **must** follow this four-section structure exactly:

### The four sections

```
[① 全局基础设定段]
@图1 <角色1 名称 + 简短描述>，@图2 <角色2 名称 + 简短描述>，@图3 <场景 名称 + 简短描述>。

[② 时间片分镜脚本]
0-X 秒：@图N <角色名> 在 @图M <场景名> 做什么，<微表情>，<光影>，镜头<一种运镜>；
X-Y 秒：...；
Y-Z 秒：...。

[③ 音频/台词段]  (optional, include if dialogue or notable audio)
台词：「...」；音效：...；环境音：...。

[④ 画质风格与防崩坏约束段]  (mandatory)
4K 高清，细节丰富，<style words>，<color tone>，<atmosphere>；面部稳定不变形、五官清晰、人体结构正常、动作自然流畅、不僵硬、画面无卡顿、无闪烁。
```

### Hard rules (per official spec)

1. **`@图N` must be followed by the name** — never bare `@图1`, always `@图1 赵长安`. This prevents Chinese tokenizer ambiguity.
2. **One camera move per time slice** — splitting required if a shot has two moves (e.g. push + pull).
3. **No `[Image1]` / `Scene N` / `lens switch` / `[asset-xxx]`** — these are deprecated forms. Only `@图N + 名称` is allowed.
4. **Quality tail is mandatory** — `4K 高清，细节丰富` opener + all seven anti-collapse items (`面部稳定不变形、五官清晰、人体结构正常、动作自然流畅、不僵硬、画面无卡顿、无闪烁`) at the close.
5. **All dialogue and audio must be preserved** — cannot be silently dropped during translation.
6. **Chinese unless dialogue is English** — `0-1 秒` not `0-1s`, `镜头` not `camera`.

### Time slice translation from `sceneDesc`

Each `镜头N[start-end s]: <description>` in `sceneDesc` becomes one `start-end 秒：<Chinese description>；` time slice in `seedancePrompt`. Translation should:
- Convert `[0-1.5s]` → `0-1.5 秒` (Chinese 秒 + space)
- Carry over the six dimensions from the shot description, but **collapse the camera move** to exactly one
- Replace `@图片N` (used in sceneDesc) with `@图N 名称` (used in Seedance)
- Use `；` between slices, `。` at the end

### `references` array shape

```json
[
  {"tag": "@图1", "name": "赵长安", "file": "zhao_1.jpg"},
  {"tag": "@图2", "name": "黑衣刺客", "file": "assassin_1.jpg"},
  {"tag": "@图3", "name": "竹林空地", "file": "url_scene_1.jpg"}
]
```

The `name` field must match exactly what appears after `@图N` in the prompt text.

## Rhythm analysis

No fixed `N = round(time/1.5)`. Read `originalText` semantically:

- **High action density** (5 verbs/sentence) → 5-6 dense shots, each 0.8-1.5s
- **Static dialogue** → 1-2 long shots, each 3-5s
- **Emotional turn** → slow-quick-freeze rhythm curve
- **Environmental description** → 2-3 free shots
- **System UI alert** → short shots, UI + reaction combo

See `references/rhythm-rules.md` for detailed cases.

**Hard constraints (always enforce):**
- Each shot ≥ 0.8s and ≤ 5s (8s max for special long takes)
- Total duration = `time` field exactly
- Continuous time (no gaps between shots)
- Strict v3.2 format (镜头N[start-end s]: ...)
- All six dimensions per shot

## Reference files

Load these as needed; don't load all upfront:

- **`references/rhythm-rules.md`** — Four-dimension rhythm analysis with worked examples.
- **`references/platform-adaptations.md`** — Detailed prompt templates, dos and don'ts, and translation examples for all five platforms.
- **`references/shot-library.md`** — 60+ camera shot library (A01-I12) with selection algorithm and emotion mapping.
- **`references/micro-expressions.md`** (NEW in v3.2) — The 50-entry library extracted for quick reference. Same content as §9.2 in the full prompt, but on-demand-loadable.
- **`references/input-schema.md`** — Expected input format and validation rules.
- **`references/full-prompt-v3.2.md`** — Complete v3.2 prompt document. Load if the user asks "what is the full system prompt" or wants to inspect/modify the underlying logic.

## Common pitfalls

1. **Reverting to v3.1 `△[time]` format** — v3.2 strict format is non-negotiable. Self-check before every row.
2. **Silently omitting a dimension** — All six required even when sparse. Write 静止 / 无运镜 / 无动作 explicitly if not applicable.
3. **Inventing micro-expressions when §9.2 covers it** — Library is more cinematic than ad-hoc. Browse it first.
4. **Forgetting `Audio:` in Veo prompts / 音效段 in Seedance** — Both have native audio. Veo: add an `Audio:` segment in English. Seedance: include the 音频 segment in Chinese (台词 / 音效 / 环境音), or weave audio into each time slice.
5. **Using English in Seedance prompts** — v3.2 mandates Chinese for Seedance per official spec. The only exception: when the original dialogue is itself in English, keep that line as-is.
6. **Using `[Image1]` or `Scene 1 / lens switch` in Seedance** — Both are deprecated as of v3.2. Use `@图N + 名称` references and time slices `0-X 秒：...；` instead.
7. **`@图N` without a following name in Seedance** — Always anchor: `@图1 赵长安`, never bare `@图1`. Prevents tokenizer ambiguity.
8. **Multiple camera moves in one Seedance time slice** — Hard prohibition per official spec. If a shot has 推入 + 拉远, split into two time slices.
9. **Missing 4K + anti-collapse tail in Seedance** — Every Seedance prompt must end with `4K 高清，细节丰富...面部稳定不变形、五官清晰、人体结构正常、动作自然流畅、不僵硬、画面无卡顿、无闪烁。` All seven anti-collapse items required.
10. **Using Chinese in HappyHorse element names** — Stick to pinyin or English for HappyHorse `name` fields.
11. **Forgetting `multi_prompt` for Kling action sequences** — When a row has 4+ shots and targeting Kling, also produce `multi_prompt` (up to 6 shots).
12. **Mismatched time totals** — Always verify Σ(shot durations) === `time` field before outputting.
13. **Repeating `@图片1` for scenes** — Scene marked only in the first shot of the row.
14. **Including scene name in `imageRefs`** — Only characters and items, never scenes.
15. **Abstract emotion words** — Never write "悲伤" or "happy" in sceneDesc; always physicalize.
16. **Writing `null` for unselected platforms** — Just omit the field entirely.
17. **Mis-resolving platform aliases** — "可灵" → `kling`, "即梦" → `seedance`, "快马" → `happyhorse`.

## Quick example (v3.2 format)

**Input:**
```json
{
  "time": 4,
  "attribute": "人物·李雷",
  "originalText": "他看到她递过来的旧信物，眼眶突然就湿了。"
}
```

**Rhythm analysis:** Low action density + emotional turn + 2 visual points → 2 long shots.

**Micro-expression library application:** #46 看到旧物 + #07 失而复得 (stacked).

**Output (abbreviated):**
```json
{
  "time": 4,
  "sceneDesc": "镜头1[0-1.5s]: 近景，李雷@图片2 站在庭院夜景@图片1 里看着对方递过来的旧玉佩@图片3，目光落在物件上不动眉眼迅速柔软又泛酸，灯笼昏黄暖光从右侧打亮面部左半边在阴影里，镜头微微跟随玉佩落下的轨迹缓慢推入，落叶沙沙声与远处隐约的虫鸣 镜头2[1.5-4s]: 特写，李雷指腹轻轻摩挲玉佩边缘嘴角像要笑眼眶却先湿，眼眶瞬间湿润瞳孔放大嘴角想笑却先发抖鼻尖微红手伸到半空停住像怕一碰就消失，灯笼暖光打在玉佩上反射出温润绿光，固定长镜头不动，环境音放大轻微呼吸声与心跳般的鼓点",
  "imageRefs": ["李雷", "旧玉佩"],
  "veoPrompt": {...},
  "grokPrompt": {...},
  "seedancePrompt": {...},
  "happyhorsePrompt": {...},
  "klingPrompt": {...},
  "prompt": ""
}
```

**Expanded seedancePrompt (showing v3.2 official four-section format):**
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

Notice in `seedancePrompt.prompt`:
- Four sections: 全局段 / 时间片段 / 环境音段 / 画质段, separated by `\n\n`
- Every `@图N` is anchored with a name (`@图1 李雷`, never bare `@图1`)
- Time slices use Chinese `0-1.5 秒：` format with full-width 秒 + space
- Quality tail ends with all seven anti-collapse items
- One camera move per slice (`缓慢推入` in slice 1, `固定长镜头不动` in slice 2)

Notice in `sceneDesc`:
- Both shots use `镜头N[start-end s]:` strictly.
- Each shot has all six dimensions present.
- Micro-expressions come directly from §9.2 library entries #46 and #07.

## Quick example with platform subset

**User request:** "只生成 Kling 和 Seedance 的，input 如下..."

**Resolved platforms:** `["seedance", "kling"]`

**Output (same row, abbreviated):**
```json
{
  "time": 4,
  "attribute": "人物·李雷",
  "originalText": "他看到她递过来的旧信物，眼眶突然就湿了。",
  "sceneDesc": "镜头1[0-1.5s]: 近景，李雷@图片2 ...（同上）镜头2[1.5-4s]: 特写，李雷指腹 ...（同上）",
  "imageRefs": ["李雷", "旧玉佩"],
  "seedancePrompt": {...},
  "klingPrompt": {...},
  "prompt": ""
}
```

`veoPrompt`, `grokPrompt`, `happyhorsePrompt` are **completely absent** — not null, not `{"skip": true}`, just omitted.

## Confirming the platform list (preamble)

Before streaming the JSONL, state which platforms will be generated. One short sentence, no commentary on script itself:

```
> Generating for: Kling, Seedance.
{...row 1...}
```

Or in Chinese if the user wrote Chinese:

```
> 仅生成 Kling、Seedance 的提示词。
{...row 1...}
```

This preamble is the ONLY commentary allowed; everything after is pure JSONL.

## Streaming behavior

After the one-line preamble, output JSONL row-by-row as you process. Do not buffer the whole output. Do not wrap in markdown code fences. Do not add commentary between rows or after the JSONL stream.

Each line must be valid standalone JSON (no leading commas, no array brackets).
