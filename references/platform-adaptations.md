# Platform Adaptation Reference

The single most important file. Defines how to translate the canonical `sceneDesc` into each platform's native prompt format.

## Conceptual model

Every row produces ONE `sceneDesc` (director-readable Chinese storyboard with `△[time]` anchors and `@图片N` markers), then FIVE platform prompts derived from it.

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
sceneDesc: △[0-3s]修炼密室@图片1，赵长安@图片2 缓慢盘膝坐下，
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
sceneDesc: △[0-2s]近景，赵长安@图片2 紧闭双眼，眉头微微皱起，
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

**Origin:** ByteDance. **Released:** February 2026.

**Language:** Chinese or English — both work well. Chinese slightly preferred (native model).

**Reference syntax:** **`[Image1]` `[Image2]` ... `[Video1]` `[Audio1]`** — square brackets, integer index.

**Multi-angle:** No (one image per reference index), but you can use many indices.

**Multi-shot:** **YES.** Use `lens switch` keyword between scenes. Up to 9 images + 3 videos + 3 audio in one prompt.

### Prompt template

```
Scene 1: [shot with [Image1] [Image2] references],
[camera move]. Audio: [...].
lens switch.
Scene 2: [next shot description].
lens switch.
Scene 3: [...]
```

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

### Translation example

Single Scene:

```
sceneDesc: △[0-3s]修炼密室@图片1，赵长安@图片2 盘膝而坐

seedancePrompt: {
  "prompt": "Cinematic close-up. The character from [Image1] sits
             cross-legged in the stone chamber from [Image2], golden
             energy gathering, slow dolly-in, candlelight ambience.
             Audio: meditative breathing, subtle wind chimes.",
  "references": [
    {"type": "image", "file": "zhao_front.jpg", "index": 1},
    {"type": "image", "file": "chamber.jpg", "index": 2}
  ]
}
```

Multi-Scene (4 shots in one generation):

```
seedancePrompt: {
  "prompt": "Scene 1: Wide shot of [Image2], the character from
             [Image1] barely visible. lens switch.
             Scene 2: Medium shot pushing in. lens switch.
             Scene 3: Close-up of face. lens switch.
             Scene 4: Extreme close-up of glowing dantian.
             Audio: building meditative tone.",
  "references": [
    {"type": "image", "file": "zhao_front.jpg", "index": 1},
    {"type": "image", "file": "chamber.jpg", "index": 2}
  ]
}
```

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
sceneDesc: △[0-3s]近景，赵长安@图片2 紧闭双眼，眉头微皱，
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
sceneDesc: △[0-3s]修炼密室@图片1，赵长安@图片2 盘膝而坐，
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
| Seedance | `[Image1]` | `references: [{type, file, index}]` |
| HappyHorse | `@name_in_pinyin` | `elements: [{name, desc, element_input_urls: [4 urls]}]` |
| Kling | `@中文名` | `elements: [{name, desc, element_input_urls: [4 urls]}]` |

## Language assignment quick rule

- **English-only**: Veo, Grok, HappyHorse
- **Chinese-only**: Kling
- **Either**: Seedance (prefer Chinese for Chinese-language drama)

## When a platform should be skipped

If the row genuinely doesn't fit a platform, output:

```json
"<platform>Prompt": {"skip": true, "reason": "<concise English reason>"}
```

Example: A 12-second row with dense action exceeds Grok Imagine's 10-second cap. Mark `grokPrompt: {"skip": true, "reason": "exceeds 10s duration cap"}` or split into two Grok calls.
