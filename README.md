<div align="center">

# Video Prompt Skill

**One drama script row → five AI video platform prompts, generated in a single pass.**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Claude Skill](https://img.shields.io/badge/Claude-Skill-8a4fff)](https://docs.claude.com/en/docs/claude-code/skills)
[![Version](https://img.shields.io/badge/version-3.2-blue)](#version-history)
[![中文](https://img.shields.io/badge/lang-中文-red)](README.zh-CN.md)

</div>

---

## 30-second pitch

Feed it one row of structured script (duration, character, dialogue). It outputs **5 prompts in 5 different syntaxes/languages** — one each for Veo 4, Grok Imagine, Seedance 2.0, HappyHorse 1.0, and Kling 3.0 Omni — ready to paste into each platform.

It does **not** generate the video and does **not** call any API — it just builds a spec-compliant bridge from "script" to "five platforms."

```
                                  ┌────► Veo 4 prompt (English, ordered array)
                                  ├────► Grok prompt (English, @image1)
script (tableData) ────► sceneDesc ────► Seedance prompt (Chinese 4-section)
+ assets (assetLists)   (6 dims)  ├────► HappyHorse prompt (English, multi-angle elements)
                                  └────► Kling prompt (Chinese, physics keywords)
```

```bash
git clone https://github.com/liqi21cn/video-prompt-skill.git \
  ~/.claude/skills/video-prompt-skill
```

Then drop structured script into Claude — it auto-triggers.

---

## ✅ What it does

| Capability | What that means |
|---|---|
| **5 platforms in one pass** | Veo 4 / Grok Imagine / Seedance 2.0 / HappyHorse 1.0 / Kling 3.0 Omni, each in native syntax |
| **Platform subset** | "Only Kling", "just Veo and Seedance", "skip Grok" all parsed. Non-selected platform fields are omitted entirely |
| **Semantic rhythm analysis** | 6s meditation → 1 long shot, 6s sword fight → 5 dense shots. No `N = time / 1.5` formula |
| **Six required dimensions** | Every shot must include shot-size / action / micro-expression / lighting / camera / audio. Self-check before output |
| **50-entry micro-expression library** | Scene-tagged presets from "suppressed grievance" to "probing villain"; copy-paste beats inventing |
| **60+ shot library** | A01–I12 codes, emotion → shot lookup |
| **Physics keywords (Kling)** | Auto-injects 真实重力 / 重心转移 / 头发惯性 for realism |
| **Official spec compliance** | Seedance follows ByteDance's `byted-ark-seedance-pe v1.0` (4-section + 7-item anti-collapse tail) |
| **Structured output** | Streaming JSONL — one row per line, pipe directly into your parser |
| **Multi-angle handling** | HappyHorse / Kling auto-use 4 angles; Veo / Grok auto-pick frontal only |
| **Dialogue lip-sync trigger** | Detects quoted dialogue; auto-adds `直视镜头，缓缓开口说道：` for Kling lip-sync |
| **Audio segment auto-fill** | Veo gets `Audio:` segment; Seedance gets 环境音 / 台词 / 音效 section |

---

## ❌ What it does NOT do

Be clear about scope so you don't have wrong expectations:

| Out of scope | Why / alternative |
|---|---|
| ❌ **Doesn't generate the video itself** | Only produces prompts — you still need to run them on Veo / Kling / etc. |
| ❌ **Doesn't call any API** | Output is text. No automatic platform API invocation |
| ❌ **Doesn't validate image URLs** | Assumes provided URLs are reachable — no fetching |
| ❌ **Can't auto-trigger outside Claude** | Reference docs readable manually, but the auto-trigger flow needs Claude Code |
| ❌ **Only these 5 platforms** | Sora, Runway, Pika, Hailuo, Vidu not supported (PRs welcome) |
| ❌ **Won't write a script from an idea** | You need the script first — skill translates script → prompt, doesn't invent plot |
| ❌ **No image/video input** | Inputs are text scripts + image URL strings only |
| ❌ **Weak cross-row continuity** | Each `tableData` row is processed independently — no auto character ID continuity |
| ❌ **No post-production** | No subtitling, color grading, editing, voicing, or compositing |
| ❌ **No negative-prompt witchcraft** | Uses sensible defaults; won't hand-tune to your genre |

---

## 🆚 vs. other approaches

| Dimension | **This skill** | Asking ChatGPT/Claude directly | Platform's built-in helper (e.g. Kling auto-prompt) | Manual writing | Single-platform template library |
|---|---|---|---|---|---|
| Multi-platform in one shot | ✅ 5 at once | ❌ 5 separate questions | ❌ that platform only | ❌ tedious | ❌ that platform only |
| Spec adherence | ✅ Strict (v3.2 self-check) | ⚠️ Stale platform knowledge | ✅ Own spec | ⚠️ Easy to miss | ⚠️ Rigid templates |
| Cross-platform semantic consistency | ✅ Same `sceneDesc` derived | ❌ Hard to guarantee | ❌ Impossible | ⚠️ Depends on author | ❌ |
| Rhythm decision | ✅ 4-dim semantic | ⚠️ Usually single-shot | ⚠️ Usually single-shot | ✅ Depends on author | ❌ |
| Micro-expression / shot library | ✅ 50 + 60 entries | ❌ Ad-hoc | ❌ Usually missing | ✅ Depends on author | ⚠️ Generic |
| Physics keywords (Kling) | ✅ Built-in | ❌ Usually missing | ⚠️ Own version | ⚠️ Requires expertise | ❌ |
| Seedance official 4-section | ✅ Enforced | ❌ Often uses old format | ✅ | ⚠️ Easy to violate | ⚠️ |
| Structured JSONL output | ✅ | ❌ Markdown prose | ❌ Web UI only | ❌ | ⚠️ |
| Open source / customizable | ✅ MIT | ❌ | ❌ Black box | ✅ Yours | ⚠️ Depends |
| Speed | ⚡ One conversation | 🐢 5 conversations | 🐢 Per-platform manual | 🐢🐢 | ⚡ But one platform |
| Batch production | ✅ Streaming JSONL | ❌ Hard | ❌ Impossible | ❌ | ⚠️ |

**TL;DR:**
- Just trying things out → use the platform's built-in helper (one platform)
- Occasional writing → manual + docs
- **Batch production / multi-platform A/B / consistent style → this skill**
- Already have a workflow → use [`references/`](references/) as a reference library

---

## 🚀 Quick start

### 1. Install

```bash
# user-level (across all projects)
git clone https://github.com/liqi21cn/video-prompt-skill.git \
  ~/.claude/skills/video-prompt-skill

# OR project-level (current project only)
git clone https://github.com/liqi21cn/video-prompt-skill.git \
  .claude/skills/video-prompt-skill
```

### 2. Verify

Restart Claude Code → ask *"list my available skills"* → you should see `video-prompt`.

### 3. Use

```
Convert this script row into Kling and Seedance prompts:

{
  "platforms": ["kling", "seedance"],
  "tableData": [
    { "time": 4, "attribute": "人物·李雷",
      "originalText": "他长叹一声，转身离去。" }
  ],
  "assetLists": {
    "characters": [{
      "name": "李雷", "description": "young man, casual modern clothes",
      "multi_angle_urls": ["url1", "url2", "url3", "url4"]
    }]
  }
}
```

Auto-triggers on phrases like *storyboard / 分镜 / video prompt / Kling / Veo / Grok / Seedance / HappyHorse / 即梦 / 可灵*.

---

## 🎯 What happens to each script row

```
1. Read originalText → 4-dimension rhythm analysis
   ├─ Action density   (how many explosive verbs?)
   ├─ Emotional arc    (calm / tension / climax / turn?)
   ├─ Dialogue?        (spoken lines get ≥2s each)
   └─ Information      (how many distinct visual points?)
       │
       ▼
2. Decide shot count + per-shot duration
   ├─ Each shot ≥ 0.8s and ≤ 5s
   └─ Σ durations = `time` exactly
       │
       ▼
3. Write sceneDesc (six dimensions per shot)
   镜头N[start-end s]: ①size ②action ③micro-expr ④lighting ⑤camera ⑥audio
   (③ should pull from the 50-entry library)
       │
       ▼
4. Derive 5 platform prompts
   ├─ Veo:        English + ordered image array
   ├─ Grok:       English + @image1
   ├─ Seedance:   Chinese 4-section + @图N+name + anti-collapse tail
   ├─ HappyHorse: English + @pinyin + multi-angle elements
   └─ Kling:      Chinese + @中文名 + physics keywords
       │
       ▼
5. Run 8-item pre-output self-check
   □ Every shot starts with 镜头N[start-end s]: ?
   □ All 6 dimensions present?
   □ Σ durations = `time`?
   □ …
       │
       ▼
6. Stream JSONL output, one row per line
```

---

## 📝 Complete example

**Input (4-second emotional beat):**

```json
{
  "tableData": [
    { "time": 4, "attribute": "人物·李雷",
      "originalText": "他看到她递过来的旧信物，眼眶突然就湿了。" }
  ],
  "assetLists": {
    "characters": [{ "name": "李雷", "description": "young man, casual modern clothes",
                     "multi_angle_urls": ["lilei_1.jpg", "lilei_2.jpg"] }],
    "items":      [{ "name": "旧玉佩", "description": "jade pendant, carved edges, worn",
                     "multi_angle_urls": ["jade_1.jpg"] }]
  }
}
```

**Skill's internal reasoning:**
- Rhythm → low action + emotional turn + 2 visual points → 2 long shots
- Micro-expression library → #46 看到旧物 + #07 失而复得 (stacked)

**Intermediate `sceneDesc`:**

```
镜头1[0-1.5s]: 近景，李雷@图片2 站在庭院夜景@图片1 里看着对方递过来的旧玉佩@图片3，
              目光落在物件上不动眉眼迅速柔软又泛酸，灯笼昏黄暖光从右侧打亮面部
              左半边在阴影里，镜头微微跟随玉佩落下的轨迹缓慢推入，
              落叶沙沙声与远处隐约的虫鸣

镜头2[1.5-4s]: 特写，李雷指腹轻轻摩挲玉佩边缘嘴角像要笑眼眶却先湿，
              眼眶瞬间湿润瞳孔放大嘴角想笑却先发抖鼻尖微红手伸到半空停住像怕一碰就消失，
              灯笼暖光打在玉佩上反射出温润绿光，固定长镜头不动，
              环境音放大轻微呼吸声与心跳般的鼓点
```

**Derived Seedance prompt (v3.2 Chinese 4-section):**

```
@图1 李雷现代休闲装年轻男性，@图2 旧玉佩翠绿色雕花边缘有磨损，@图3 庭院夜景灯笼昏黄落叶满地。

0-1.5 秒：@图1 李雷站在@图3 庭院夜景里看着对方递过来的@图2 旧玉佩，
        目光落在物件上不动眉眼迅速柔软又泛酸，灯笼昏黄暖光从右侧打亮面部
        左半边在阴影里，镜头微微跟随玉佩落下的轨迹缓慢推入；
1.5-4 秒：@图1 李雷指腹轻轻摩挲@图2 旧玉佩边缘嘴角像要笑眼眶却先湿，
        眼眶瞬间湿润瞳孔放大嘴角想笑却先发抖鼻尖微红手伸到半空停住，
        灯笼暖光打在玉佩上反射出温润绿光，镜头固定长镜头不动让情绪自然流出。

环境音：落叶沙沙声、远处隐约的虫鸣、轻微呼吸声与心跳般的低沉鼓点。

4K 高清，细节丰富，都市情感剧风格，暖光低饱和柔焦质感，写实电影情感氛围；
面部稳定不变形、五官清晰、人体结构正常、动作自然流畅、不僵硬、画面无卡顿、无闪烁。
```

**Derived Kling prompt (Chinese + physics keywords):**

```
@李雷 看着对方递过来的@旧玉佩，目光落在物件上不动眉眼迅速柔软又泛酸；
随后指腹轻轻摩挲玉佩边缘，眼眶瞬间湿润瞳孔放大嘴角想笑却先发抖鼻尖微红手伸到半空停住。
灯笼昏黄暖光从右侧打亮面部左半边在阴影里，玉佩反射温润绿光。
镜头先缓慢推入聚焦玉佩，随后固定长镜头让情绪自然流出。
写实电影质感，真实重力感，呼吸节奏自然，头发惯性。
```

**Derived Veo prompt (English + Audio segment):**

```
Medium close-up of a young Asian man in modern casual clothes in a courtyard
at night, lanterns glowing warm yellow. He looks at an old jade pendant being
handed to him, gaze fixed on the object, brows softening with welling emotion.
Camera slowly pushes in following the pendant's descending arc. Then extreme
close-up: his fingertip traces the jade edge, eyes rim with tears, hand pauses
mid-reach as if afraid to touch. Locked long take, warm low-saturated tone,
cinematic emotional drama style.
Audio: rustling leaves, distant insects, soft breath, heartbeat-like low drum.
```

**Note:** the same scene produces 5 syntactically different prompts (`@图N + name`, `@image1`, `@li_lei`, `@李雷`, plain description) — but the semantics / emotion / shot composition stay consistent.

---

## 🛠️ Platform cheatsheet

| Platform | Language | Ref syntax | Multi-shot | One-line characterization |
|---|---|---|---|---|
| **Veo 4** | English | ordered array (no markers) | no | Best prompt adherence, native audio |
| **Grok Imagine** | English | `@image1` | frame-chain | Fast iteration, viral social-media feel |
| **Seedance 2.0** | **Chinese** | `@图N + name` | **time slices** | Best multi-shot, aligned to Volcengine spec |
| **HappyHorse 1.0** | English | `@element_name` | shot array | Best single-portrait, multi-angle elements |
| **Kling 3.0 Omni** | **Chinese** | `@中文名` | `multi_prompt` | Best physics, best action choreography |

Per-platform pitfalls and templates: [`references/platform-adaptations.md`](references/platform-adaptations.md).

---

## ⚙️ Advanced features

### v3.2 strict shot format

Every shot must be `镜头N[start-end s]:` — Arabic numerals, half-width symbols. These are all errors:

| ❌ Wrong | Why |
|---|---|
| `△[0-2s] xxx` | Missing `镜头N` + colon |
| `0-2秒：xxx` | Missing brackets, Chinese 秒 |
| `(0-1.5s) xxx` | Round parens |
| `第1段 0-1.5s xxx` | `第N段` not allowed |
| `Shot1[0-1.5s]: xxx` | English `Shot` |
| `镜头一[0-1.5s]: xxx` | Chinese numeral |

### Six required dimensions

| # | Dimension | Required element |
|---|---|---|
| ① | **Shot size** | 远景 / 全景 / 中景 / 近景 / 特写 / 极近特写 |
| ② | **Subject + action** | who + what + how |
| ③ | **Micro-expression** | eye / brow / mouth / breath / small action (pull from [`micro-expressions.md`](references/micro-expressions.md)) |
| ④ | **Lighting** | direction + temperature + intensity + shadows |
| ⑤ | **Camera move** | push / pull / pan / tilt / track / rotate + speed |
| ⑥ | **Audio** | ambient / action / emotional SFX |

For non-applicable dimensions (e.g. a frozen still has no camera move), write `静止 / 无运镜 / 无动作` explicitly — never silently omit.

### 50-entry micro-expression library (new in v3.2)

| Group | Entries | Use cases |
|---|---|---|
| 委屈/隐忍 (Restraint) | 6 | misunderstood / scapegoated / forced farewell |
| 紧张/慌乱/心虚 (Anxiety) | 5 | exposed secret / hesitation / breakdown |
| 心动/重逢/情感 (Affection) | 7 | first sight / reunion / seeing an old keepsake |
| 冷淡/克制/距离感 (Aloofness) | 4 | fake indifference / aristocratic distance |
| 怒意/反击/不服 (Anger) | 4 | suppressed rage / cold counter / battle-scar smile |
| 权谋/试探/隐忍 (Political) | **11** | political tension / fake calm / killing-intent flash |
| 崩溃/疯感/迷失 (Collapse) | 3 | obsession collapse / memory waking |
| 强者气场/神性/威压 (Authority) | 4 | first-love filter / sect master / divine sorrow |
| 强忍/坚守/信念 (Endurance) | 3 | bad news / misunderstanding cleared |
| 释然/结局/守护 (Resolution) | 3 | survival aftermath / silent guardian / ending peace |

Each entry follows: **eye landing + brow change + mouth change + breath pause + small action + scene tag**. Full list: [`micro-expressions.md`](references/micro-expressions.md).

### Seedance 4-section (v3.2 aligned to Volcengine official spec)

```
[① Global setup]    @图1 name+desc, @图2 ……。
[② Time slices]     0-X 秒：@图N in @图M doing what, <6 dims>;
[③ Audio segment]   台词：「…」；音效：…；环境音：…。
[④ Anti-collapse]   4K 高清，<style>；面部稳定不变形、五官清晰、
                   人体结构正常、动作自然流畅、不僵硬、画面无卡顿、无闪烁。
```

**Hard rules:** `@图N` must be followed by a name (never bare `@图1`); one camera move per time slice; all seven anti-collapse items required.

### Kling physics keywords

| Keyword | Effect |
|---|---|
| `真实重力` | Gravity-correct motion |
| `重心转移` | Realistic weight shift |
| `脚跟先落地` | Heel-first natural walking |
| `布料下垂` | Natural fabric drape |
| `头发惯性` | Hair momentum in turns |
| `液体粘度` | Realistic fluid viscosity |

### Shot library (60+ shots, A01–I12)

9 categories: A push/pull · B trajectory · C speed · D angle · E close-up · F psychology · G action · H transition · I effect. Emotion → shot code lookup in [`references/shot-library.md`](references/shot-library.md).

---

## 📂 Project structure

```
video-prompt-skill/
├── SKILL.md                          # Entry — schema, workflow, pitfalls
├── README.md                         # This file
├── README.zh-CN.md                   # 中文版
├── LICENSE                           # MIT
└── references/                       # ↓ all loaded on demand, not upfront
    ├── input-schema.md               # Input format + validation
    ├── rhythm-rules.md               # 4-dim rhythm analysis (5 worked examples)
    ├── shot-library.md               # 60+ shots (A01–I12) + emotion lookup
    ├── platform-adaptations.md       # Per-platform guide (v3.2 Seedance rewrite)
    ├── micro-expressions.md          # 🆕 v3.2: 50 micro-expression presets
    └── full-prompt-v3.2.md           # Full v3.2 system prompt archive
```

---

## ❓ FAQ

<details>
<summary><b>Q: I don't have structured JSON — just plain narrative. Can it still help?</b></summary>

Yes. The skill helps you format raw text into `tableData` first, then proceeds. You just need to provide the text + character / scene reference URLs.
</details>

<details>
<summary><b>Q: What if my character isn't in assetLists?</b></summary>

Skill describes by name only — no `@` reference. All platform prompts fall back to text. Skill flags `"_no_ref": ["X"]` so you know.
</details>

<details>
<summary><b>Q: How strict is `time`? I want exactly 6s — not 5.8.</b></summary>

The skill verifies Σ(shot durations) === `time` exactly. Each shot is ≥ 0.8s and ≤ 5s. If the rhythm can't fit, the skill warns.
</details>

<details>
<summary><b>Q: Can I use this without Claude?</b></summary>

The auto-trigger flow needs Claude Code. But `references/` files are usable as a manual prompt-engineering guide — copy micro-expression entries or shot codes directly.
</details>

<details>
<summary><b>Q: How do I update?</b></summary>

```bash
cd ~/.claude/skills/video-prompt-skill && git pull
```
</details>

<details>
<summary><b>Q: Can I add my own platform (e.g. Sora)?</b></summary>

Fork →
1. Add a section to `references/platform-adaptations.md`
2. Add the platform ID to the canonical list in `SKILL.md`
3. Update the output schema with a `<your>Prompt` field
4. Open a PR
</details>

<details>
<summary><b>Q: I have a v3.1 downstream pipeline expecting △[time] and [Image1]. How do I migrate?</b></summary>

v3.2 is a breaking format change. Update your parsers to expect:
- `sceneDesc` uses `镜头N[start-end s]:`, not `△[time]`
- `seedancePrompt.prompt` is Chinese 4-section, not `[Image1] + Scene N`
- `seedancePrompt.references` shape: `{tag, name, file}` (was `{type, file, index}`)

Schema **field names are unchanged** — only the *content* of `sceneDesc` and `seedancePrompt.prompt` changed.
</details>

<details>
<summary><b>Q: Does it fetch image URLs to check they exist?</b></summary>

No. URL reachability is downstream — skill just passes URLs through. Use any image host.
</details>

---

## 📜 Version history

| Area | v3.0 | v3.1 | **v3.2** |
|---|---|---|---|
| Shot count | `N = round(time / 1.5)` | Semantic 4-dim | Same as v3.1 |
| Platform subset | ❌ Always 5 | ✅ NL or JSON | Same as v3.1 |
| Skipped platform output | — | Completely omitted | Same as v3.1 |
| Shot format | `△[time]` | `△[time]` | **`镜头N[start-end s]:` strict** |
| Per-shot dimensions | Free-form | Free-form | **6 required** |
| Micro-expressions | Ad-hoc | Ad-hoc | **50-entry library** |
| Seedance format | `[Image1]` English | Same as v3.0 | **Chinese 4-section + `@图N+name`** |
| Pre-output self-check | Implicit | Implicit | **8-item checklist** |
| Shot library | 40 | 60+ | Same as v3.1 |
| Physics keywords | — | Kling-specific | Same as v3.1 |

Release notes: [v3.2.0](https://github.com/liqi21cn/video-prompt-skill/releases/tag/v3.2.0)

---

## 🤝 Contributing

Issues and PRs welcome.

Good first PRs:
- Add a new shot to `references/shot-library.md`
- Add a new micro-expression entry to `references/micro-expressions.md`
- Translate `references/*.md` to another language
- Add a recently-discovered platform behavior to `references/platform-adaptations.md`

For substantial changes (new platforms, rhythm algorithm, format constraints), open an issue first.

---

## 📄 License

[MIT](LICENSE) — use freely in commercial and personal projects.

---

<div align="center">

**Found this useful?** ⭐ Star the repo so others can find it.

[Issues](https://github.com/liqi21cn/video-prompt-skill/issues) · [Releases](https://github.com/liqi21cn/video-prompt-skill/releases) · [中文](README.zh-CN.md)

</div>
