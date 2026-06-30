# Platform Adaptation Reference

The single most important file. Defines how to translate the canonical `sceneDesc` into each platform's native prompt format.

## Conceptual model

Every row produces ONE `sceneDesc` (director-readable Chinese storyboard with `镜头N[start-end s]:` prefixes and `@图片N` markers), then FIVE platform prompts derived from it.

The translation isn't just word-swapping — each platform has its own:
- Language preference (Chinese vs English)
- Reference syntax (`@image1`, `[Image1]`, `@element_name`, or no markers)
- Number of reference images per entity (1 vs 2-4 multi-angle)
- Multi-shot support (none vs `lens switch` vs `multi_prompt`)

---

## Platform 1: Veo 4

**Origin:** Google DeepMind. **Released:** 2026.

**Language:** English (Veo 4 is English-native; Chinese works but quality drops).

**Reference syntax:** **None.** Veo does not use textual reference markers. Instead, you pass reference images as an ordered array (max 3). The model identifies subjects from the images, not from the prompt.

**Multi-angle:** No. Use only the clearest frontal photo per entity.

**Multi-shot:** No. One prompt = one continuous clip (4, 6, or 8 seconds typical).

### Prompt template

```
[Shot type], [Subject doing action] [in/at setting],
[lighting/atmosphere], [style/lens], [camera movement].
Audio: [ambient sounds, dialogue if any].
```

### Key strengths

- Most disciplined prompt adherence among the five
- Best handling of subtle motion (micro-expressions, breathing)
- Strongest cinematography vocabulary (anamorphic, dolly, 35mm, Steadicam)
- Native audio generation including dialogue
- Static product/commercial shots

### Key weaknesses / pitfalls

- **Don't write precise action timings** ("walks for 3 seconds then turns") — Veo will fail
- **Don't write multiple story arcs** in one prompt
- **Don't forget the `Audio:` segment** — wastes Veo's native audio
- Asian face details sometimes less refined than HappyHorse/Kling
- 3-image reference cap is a real limit (no scaling up)

### Translation example

```
sceneDesc: 镜头1[0-3s]: 修炼密室@图片1，赵长安@图片2 缓慢盘膝坐下，
          脊柱挺直，眼睑缓缓合拢，胸腔随呼吸缓缓起伏

veoPrompt: {
  "prompt": "Medium shot in a dimly lit stone meditation chamber.
             A young cultivator in dark robes slowly sits cross-legged,
             spine upright, eyelids gently closing, chest rising and
             falling with deep breaths. Candlelight flickering on walls,
             anamorphic lens, 35mm film grain, slow dolly-in.
             Audio: subtle wind chimes, deep meditative breathing.",
  "reference_images": ["zhao_front.jpg", "chamber_wide.jpg"],
  "negative_prompt": "blurry, low quality, distorted, watermark,
                      text overlay, extra fingers, deformed faces"
}
```

Note: `@图片2 赵长安` becomes "A young cultivator" in the prompt body, and the actual image is in `reference_images`.

---

## Platform 2: Grok Imagine

**Origin:** xAI. **Released:** February 2026 (Grok Imagine 1.0).

**Language:** English (Chinese works but English is more reliable).

**Reference syntax:** **`@image1` `@image2` `@image3`** — explicit markers.

**Multi-angle:** No. One frontal image per reference.

**Multi-shot:** No. But supports frame-chaining: use the last frame of generation N as the `image_urls` input to generation N+1, with prompt "continues seamlessly from previous shot".

### Prompt template

```
@image1 [action verb] [direction/manner], [style keywords],
[mood], [audio cue].
```

### Key strengths

- Social-media-ready visuals (TikTok, Instagram, X)
- Viral and meme-style aesthetics
- Strong character generation (Aurora base)
- In-image text rendering
- Fast iteration (low cost per generation)

### Key weaknesses / pitfalls

- **Hard 10-second cap** (some implementations 15s)
- **Keep prompts short** — under 100 characters works best
- **Avoid generic adjectives** ("4K", "high quality") — they're noise
- Less disciplined than Veo with prompt adherence
- Less cinematic than Kling/Veo for narrative scenes

### Translation example

```
sceneDesc: 镜头1[0-2s]: 近景，赵长安@图片2 紧闭双眼，眉头微微皱起，
          胸腔缓缓起伏

grokPrompt: {
  "prompt": "@image1 sits with eyes tightly closed, brows slightly
             furrowed, chest rising and falling, cinematic close-up,
             dim warm lighting, mystical atmosphere, deep breathing
             ambience.",
  "image_urls": ["zhao_front.jpg"],
  "duration": 2
}
```

### Special: frame-chain extension

For a 30-second sequence, generate six 5-second clips, each using the previous clip's last frame as `image_urls`:

```
{"prompt": "continues seamlessly from previous shot, character now opens his eyes", "image_urls": ["last_frame_of_clip2.jpg"]}
```

---

## Platform 3: Seedance 2.0

**Origin:** ByteDance (字节系，Volcengine 火山引擎). **Released:** February 2026.

**Language:** **Chinese only** (mandated by official Volcengine spec `byted-ark-seedance-pe v1.0`). The model is trained primarily on Chinese corpora; English prompts produce visibly weaker results. Exception: keep original English dialogue as-is.

**Reference syntax:** **`@图N` + immediately following Chinese name** (e.g. `@图1 赵长安`, `@图2 黑衣刺客`). Bare `@图N` without a name is **prohibited** — it causes Chinese tokenizer ambiguity.

**Multi-angle:** No (one image per `@图N` reference), but you can use many `@图N` indices. Same character with multiple angles becomes `@图1 赵长安正脸` + `@图2 赵长安侧脸`.

**Multi-shot:** **YES — via time slices, not lens switches.** The official structure uses Chinese time annotations like `0-3 秒：...；3-6 秒：...；6-12 秒：...。` within a single prompt. Up to 9 images + 3 videos + 3 audio total assets in one prompt.

### Mandatory four-section structure

Every Seedance prompt **must** be organized into these four sections (per official skill):

```
[① 全局基础设定段]
（声明角色/环境/资产 + 建立 @图N 到名称的映射）

[② 时间片分镜脚本]
0-X 秒：@图N 角色名 在 @图M 场景名 做什么，<微表情>，<光影>，镜头<一种运镜>；
X-Y 秒：...；
Y-Z 秒：...。

[③ 音频/台词段]（可选，有对白或显著音效时必含）
台词：「...」；音效：...；环境音：...。

[④ 画质风格与防崩坏约束段]（强制）
4K 高清，细节丰富，<风格词>，<色调>，<氛围>；面部稳定不变形、五官清晰、人体结构正常、动作自然流畅、不僵硬、画面无卡顿、无闪烁。
```

Sections separated by `\n\n` (empty line). The four-section structure is non-negotiable per official spec.

### Hard rules

| # | Rule | Why |
|---|---|---|
| 1 | `@图N` must be followed by a Chinese name | Prevents tokenizer parsing ambiguity |
| 2 | One camera move per time slice | Multi-move per slice degrades motion quality |
| 3 | No `[Image1]` / `Scene N` / `lens switch` / `[asset-xxx]` | All deprecated forms |
| 4 | Mandatory quality tail (4K + 7 anti-collapse items) | Without this, Seedance defaults to lower fidelity |
| 5 | Preserve all dialogue and audio | Cannot silently drop |
| 6 | Time format `0-1.5 秒` (Chinese 秒 + space) | Not `0-1.5s`, not `0~1.5秒` |
| 7 | Use `；` between slices, `。` at end | Punctuation matters for parsing |

### Translation from `sceneDesc` to Seedance

Each `镜头N[start-end s]: <description>` becomes one time slice:

```
镜头1[0-1.5s]: 中景，赵长安@图片2 缓慢盘膝坐下，眉心微皱呼吸放缓，烛火暖光从右侧打亮面部，缓慢推入聚焦面部，环境音衣料摩擦声
    ↓
0-1.5 秒：@图1 赵长安在@图3 修炼密室中缓慢盘膝坐下，眉心微皱呼吸放缓，烛火暖光从右侧打亮面部，镜头缓慢推入聚焦面部；
```

Notice:
- `镜头1[0-1.5s]:` → `0-1.5 秒：` (Chinese unit + space)
- `@图片2` → `@图1 赵长安` (Seedance uses its own numbering + name anchor)
- `@图片1` (scene) → `@图3 修炼密室` (scene typically becomes last `@图N`)
- The audio dimension can be moved to the dedicated 音频段 OR kept in-slice — both are valid

### Key strengths

- **Best multi-shot model** — single generation produces coherent multi-scene narrative
- Up to 15 seconds at 2K
- Native synchronized audio
- Strongest with dynamic action sequences
- Multi-asset compositing (mix character + product + scene refs)

### Key weaknesses / pitfalls

- Not as photorealistic as Veo for static product shots
- Background drift in long takes
- Lip-sync less precise than HappyHorse
- **Strict format compliance required** — deviating from the four-section structure noticeably degrades output

### Translation example — single time slice

```
sceneDesc: 镜头1[0-3s]: 中景，赵长安@图片2 在 修炼密室@图片1 中盘膝而坐，
                       眉心松开呼吸悠长，烛火暖光从两侧低位打亮，
                       缓慢推入聚焦面部，灵气流动的低频嗡鸣

seedancePrompt: {
  "prompt": "@图1 赵长安身着玄色道袍剑眉星目，@图2 修炼密室幽暗石室壁刻满符文。

0-3 秒：@图1 赵长安在@图2 修炼密室中央盘膝而坐，眉心松开呼吸悠长嘴角带极浅平静，烛火暖光从两侧低位打亮面部，镜头缓慢推入聚焦面部。

音效：灵气流动的低频嗡鸣、衣料摩擦细响、远处隐约的洞穴回声。

4K 高清，细节丰富，国风修仙写实风格，暖金色调，烛光氛围沉静神秘；面部稳定不变形、五官清晰、人体结构正常、动作自然流畅、不僵硬、画面无卡顿、无闪烁。",
  "references": [
    {"tag": "@图1", "name": "赵长安", "file": "zhao_front.jpg"},
    {"tag": "@图2", "name": "修炼密室", "file": "chamber.jpg"}
  ]
}
```

### Translation example — multi time slice (4 shots in one generation)

```
seedancePrompt: {
  "prompt": "@图1 赵长安身着玄色道袍，@图2 修炼密室幽暗石室壁刻满符文。

0-2 秒：远景@图2 修炼密室全貌，@图1 赵长安在中央几乎不可见，烛火微光闪烁，镜头静止远景观察；
2-4 秒：中景，@图1 赵长安进入画面盘膝而坐，呼吸放缓眉心松开，烛火暖光打亮面部轮廓，镜头缓慢推入；
4-6 秒：近景，@图1 赵长安面部细节，眉心松开嘴角带极浅平静，烛火在瞳孔反射出温暖光点，镜头继续推入；
6-8 秒：极近特写，@图1 赵长安丹田位置浮现金色灵气，灵气如潮水般涌入身体，金光在肌肤上流动，镜头微距聚焦丹田。

音效：低频嗡鸣由弱到强、衣料摩擦细响、灵气流动的清亮共鸣。

4K 高清，细节丰富，国风修仙写实风格，暖金色调与符文蓝光交织，沉静神秘的渐入式氛围；面部稳定不变形、五官清晰、人体结构正常、动作自然流畅、不僵硬、画面无卡顿、无闪烁。",
  "references": [
    {"tag": "@图1", "name": "赵长安", "file": "zhao_front.jpg"},
    {"tag": "@图2", "name": "修炼密室", "file": "chamber.jpg"}
  ]
}
```

Notice the **渐入式** flow: 远景 → 中景 → 近景 → 极近特写, each in its own time slice with one camera move. This is the canonical Seedance pattern.

### Common mistakes to avoid

- ❌ Writing in English: `Scene 1: A young man sits...`
- ❌ Bare reference: `@图1 sits cross-legged...` (missing name after `@图1`)
- ❌ Multi-camera per slice: `0-2 秒：...镜头推入再拉远...`
- ❌ Missing quality tail: ends with the time slices, no `4K 高清`
- ❌ Incomplete anti-collapse: only 3-4 of the 7 items
- ❌ Using `lens switch` to separate scenes: this is deprecated
- ❌ Asset IDs in description: `Scene 2: [asset-zhao-changan] enters...`

---

## Platform 4: HappyHorse 1.0

**Origin:** Alibaba (open-source). **Released:** April 2026.

**Language:** English (best results). Chinese works for descriptions but element names should be pinyin or English.

**Reference syntax:** **`@element_name`** — but the name is YOUR choice, defined in the `elements` array. The element bundles 2-4 multi-angle images.

**Multi-angle:** **YES — this is HappyHorse's superpower.** Supply 2-4 angles of the same subject per element for maximum consistency.

**Multi-shot:** Yes, via shot array (up to 5 shots, each with prompt + duration).

### Prompt template

```
@element_name [action], [emotion via physical description],
[lighting], [audio].
```

Plus, in the API payload:

```json
"elements": [
  {
    "name": "zhao_changan",
    "description": "young Asian male cultivator, sharp features,
                    long black hair, dark Tang-style robes",
    "element_input_urls": [
      "<front view>",
      "<side view>",
      "<three-quarter view>",
      "<back view>"
    ]
  }
]
```

### Key strengths

- **Best single-character portrait quality** in current benchmarks
- **Best facial expression and skin texture**
- **Multi-angle elements** make character consistency exceptional
- Native multilingual lip-sync (Mandarin, Cantonese, English, Japanese, Korean, German, French)
- Open-source — can self-host

### Key weaknesses / pitfalls

- **Element name must be pinyin or English** — Chinese unicode in `@` references can fail
- Less strong with complex multi-character action than Kling
- Multi-shot capped at 5 shots
- Description field heavily affects identity — write it carefully

### Translation example

```
sceneDesc: 镜头1[0-3s]: 近景，赵长安@图片2 紧闭双眼，眉头微皱，
          胸腔缓缓起伏

happyhorsePrompt: {
  "prompt": "@zhao_changan sits cross-legged with eyes closed,
             eyebrows slightly furrowed, chest rising and falling
             with deep breaths, golden warm side lighting, faint
             sound of meditative breathing.",
  "elements": [
    {
      "name": "zhao_changan",
      "description": "young Asian male cultivator, sharp features,
                      long black hair, dark Tang-style robes",
      "element_input_urls": [
        "zhao_front.jpg",
        "zhao_side.jpg",
        "zhao_three_quarter.jpg",
        "zhao_back.jpg"
      ]
    }
  ]
}
```

The four images per element are the key to HappyHorse's consistency advantage.

---

## Platform 5: Kling 3.0 Omni

**Origin:** Kuaishou (快手). **Released:** February 2026.

**Language:** **Chinese (strongly preferred)** — Kling is Chinese-native and outperforms in Chinese.

**Reference syntax:** **`@中文名`** — Chinese element names work directly.

**Multi-angle:** **YES — 4+ multi-angle per element recommended.**

**Multi-shot:** **YES, via `multi_prompt`** — up to 6 shots, each with own prompt and duration. This is the most powerful multi-shot system of the five.

### Prompt template

```
@元素名 [动作过程化描述]，[运镜词]，[物理细节关键词]，
[光影方案]，[风格质感]。
```

Plus, in the API payload:

```json
"elements": [
  {
    "name": "赵长安",
    "description": "黑发青年，剑眉星目，玄色道袍",
    "element_input_urls": ["正面.jpg", "侧面.jpg", "背面.jpg", "四分之三.jpg"]
  }
],
"multi_prompt": [
  {"prompt": "@赵长安 缓慢推门入室", "duration": 3},
  {"prompt": "@赵长安 盘膝坐下", "duration": 4}
]
```

### Key strengths

- **Best physics simulation** — realistic gravity, weight transfer, water/fabric/hair physics
- Strongest with **action choreography** (combat, chase, parkour)
- IP asset reuse via persistent elements across sessions
- Multi-character with separate dialogue control
- Voice element binding (one character has stable voice across all generations)
- Native dialogue lip-sync in multiple languages
- `multi_prompt` is the most flexible multi-shot system

### Key weaknesses / pitfalls

- Light source obedience less precise than Veo for product shots
- Less photorealistic than Veo for static commercial work
- Requires more setup time (defining elements upfront)
- Better in Chinese than English (so plan content language accordingly)

### Special: physics keywords

These keywords meaningfully improve motion realism:

| Keyword | Effect |
|---|---|
| 真实重力 | Gravity-correct falling/dropping |
| 重心转移 | Realistic weight shift in steps |
| 脚跟先落地 | Heel-first natural walking |
| 布料下垂 | Natural fabric drape |
| 头发惯性 | Hair momentum in turns |
| 液体粘度 | Realistic fluid viscosity |

### Special: dialogue trigger

```
@角色名 直视镜头，缓缓开口说道："台词内容。"
```

This triggers Kling's lip-sync system. Use `说道：「...」` rather than just quoting.

### Translation example

```
sceneDesc: 镜头1[0-3s]: 修炼密室@图片1，赵长安@图片2 盘膝而坐，
          脊柱挺直，眼睑缓缓合拢

klingPrompt: {
  "prompt": "@赵长安 缓慢盘膝坐下，脊柱挺直，双手自然搭于膝盖，
             眼睑缓缓合拢，胸腔随呼吸缓缓起伏。
             缓慢推入镜头聚焦面部，烛火质感暖光，
             真实重力感，呼吸节奏自然，写实电影质感。",
  "elements": [
    {
      "name": "赵长安",
      "description": "黑发青年，剑眉星目，玄色道袍",
      "element_input_urls": [
        "zhao_front.jpg",
        "zhao_side.jpg",
        "zhao_three_quarter.jpg",
        "zhao_back.jpg"
      ]
    }
  ]
}
```

For action sequences with 4+ shots, ALSO produce `multi_prompt`:

```json
"multi_prompt": [
  {"prompt": "@赵长安 重心下沉左脚滑出", "duration": 1},
  {"prompt": "特写@赵长安 反手握刀", "duration": 0.8},
  {"prompt": "@赵长安 长刀划出银光", "duration": 0.7}
]
```

---

## Quick-translate cheat sheet

When you have `@图片N 实体名` in `sceneDesc`, translate to:

| Platform | Body becomes | Add to payload |
|---|---|---|
| Veo | "the character" (no marker) | `reference_images: [url]` |
| Grok | `@image1` | `image_urls: [url]` |
| Seedance | `@图1 角色名` (Chinese, name required) | `references: [{tag, name, file}]` |
| HappyHorse | `@name_in_pinyin` | `elements: [{name, desc, element_input_urls: [4 urls]}]` |
| Kling | `@中文名` | `elements: [{name, desc, element_input_urls: [4 urls]}]` |

## Language assignment quick rule

- **English-only**: Veo, Grok, HappyHorse
- **Chinese-only**: Kling, **Seedance** (字节系产品，官方规范强制中文)

## When a platform should be skipped

If the row genuinely doesn't fit a platform, output:

```json
"<platform>Prompt": {"skip": true, "reason": "<concise English reason>"}
```

Example: A 12-second row with dense action exceeds Grok Imagine's 10-second cap. Mark `grokPrompt: {"skip": true, "reason": "exceeds 10s duration cap"}` or split into two Grok calls.
