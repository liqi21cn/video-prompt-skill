<div align="center">

# Video Prompt Skill

**Convert movie / drama scripts into production-ready prompts for five AI video platforms — in one pass.**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Claude Skill](https://img.shields.io/badge/Claude-Skill-8a4fff)](https://docs.claude.com/en/docs/claude-code/skills)
[![Version](https://img.shields.io/badge/version-3.1-blue)](#whats-new-in-v31)
[![中文](https://img.shields.io/badge/lang-中文-red)](README.zh-CN.md)

</div>

One drama script row in → five platform-ready prompts out. Each prompt uses the right language (Chinese for Kling, English for Veo / Grok / HappyHorse), the right reference syntax, the right multi-shot structure, and the right level of detail.

---

## Table of contents

- [Why this skill](#why-this-skill)
- [Supported platforms](#supported-platforms)
- [Quick start](#quick-start)
- [Input format](#input-format)
- [Output format](#output-format)
- [Platform selection](#platform-selection)
- [Rhythm analysis (v3.1 core)](#rhythm-analysis-v31-core)
- [Shot library](#shot-library)
- [Worked examples](#worked-examples)
- [Per-platform notes](#per-platform-notes)
- [Common pitfalls](#common-pitfalls)
- [FAQ](#faq)
- [Project structure](#project-structure)
- [What's new in v3.1](#whats-new-in-v31)
- [Contributing](#contributing)
- [License](#license)

---

## Why this skill

Generating prompts for five video AI platforms by hand is tedious and error-prone:

- **Kling** needs Chinese prompts and Chinese `@元素名` references
- **HappyHorse** needs pinyin element names and 2–4 multi-angle images per character
- **Seedance** needs `[Image1]` brackets and `lens switch` between scenes
- **Grok** needs short English prompts with `@image1`
- **Veo** doesn't use textual markers at all — just an ordered image array

This skill takes one structured script row and emits all five (or any subset) in their native format, with consistent semantics across all of them. The trick: every row first becomes a Chinese director's storyboard (`sceneDesc` with `△[time]` anchors and `@图片N` markers), then translates into each platform's syntax.

**Key features:**

- ✅ All five platforms (Veo 4, Grok Imagine, Seedance 2.0, HappyHorse 1.0, Kling 3.0 Omni)
- ✅ Subset selection by JSON field OR natural language (`"只要 Kling"`, `"skip Grok"`)
- ✅ Semantic rhythm analysis — no more `N = round(time / 1.5)` formula
- ✅ 60+ camera shot library with emotion → shot-code mapping
- ✅ Physics keywords for Kling (`真实重力`, `重心转移`, `头发惯性`)
- ✅ Native audio handling for Veo / Seedance
- ✅ Multi-shot orchestration (Seedance `lens switch`, Kling `multi_prompt`)
- ✅ Streaming JSONL output for downstream pipelines

---

## Supported platforms

| Platform | Origin | Language | Reference syntax | Imgs per ref | Multi-shot |
|---|---|---|---|---|---|
| **Veo 4** | Google DeepMind | English | none (ordered array) | 1 | no |
| **Grok Imagine** | xAI | English | `@image1` | 1 | frame-chain |
| **Seedance 2.0** | ByteDance | EN / CN | `[Image1]` | 1 | `lens switch` |
| **HappyHorse 1.0** | Alibaba | English | `@element_name` | **2–4 multi-angle** | shot array |
| **Kling 3.0 Omni** | Kuaishou | **Chinese** | `@中文名` | **4+ multi-angle** | `multi_prompt` |

**Strengths at a glance:**

- **Veo** — strongest prompt adherence, best for static / commercial / micro-expression
- **Grok** — fastest iteration, viral social-media aesthetic, in-image text rendering
- **Seedance** — best multi-shot in a single generation, native audio
- **HappyHorse** — best single-character portraits, native multilingual lip-sync
- **Kling** — best physics simulation, best action choreography, IP element reuse

---

## Quick start

### 1. Install

Drop the skill into your Claude skills directory:

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

Ask Claude with structured input:

> 帮我把这段剧本转成 Kling 和 Seedance 的提示词：
>
> ```json
> { "tableData": [...], "assetLists": {...} }
> ```

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

Three groups, each an array of asset objects:

```json
{
  "assetLists": {
    "scenes":     [{ "name", "description", "multi_angle_urls": [...] }],
    "characters": [{ "name", "description", "multi_angle_urls": [...] }],
    "items":      [{ "name", "description", "multi_angle_urls": [...] }]
  }
}
```

**`multi_angle_urls` sizing tip:**

| Targeted platforms | Recommended URL count |
|---|---|
| Only Veo / Grok | 1 frontal |
| Only Seedance | 1–3 |
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
  "sceneDesc": "△[0-2s]近景，李雷@图片2 肩膀塌落… △[2-4s]中景背影…",
  "imageRefs": ["李雷"],

  "veoPrompt":        { "prompt": "...", "reference_images": [...], "negative_prompt": "..." },
  "grokPrompt":       { "prompt": "...", "image_urls": [...], "duration": 4 },
  "seedancePrompt":   { "prompt": "...", "references": [...] },
  "happyhorsePrompt": { "prompt": "...", "elements": [...] },
  "klingPrompt":      { "prompt": "...", "elements": [...], "multi_prompt": [...] },

  "prompt": ""
}
```

**Rules:**

- Fields `time`, `attribute`, `originalText`, `sceneDesc`, `imageRefs`, `prompt: ""` always appear
- Platform fields appear **only if requested** — non-selected platforms are *completely omitted* (no `null`, no `{"skip": true}`)
- Output is one row per line — no array brackets, no markdown fences, no commentary

The trailing `"prompt": ""` is mandatory for downstream backward compatibility.

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

**If both methods are present**, the JSON field wins. The skill mentions this in a one-line preamble.

### Single-platform optimizations

When only one platform is selected, optional optimizations kick in:

- **Only Kling** — write `sceneDesc` directly in Chinese, skip the `@图片N` intermediate
- **Only Veo** — pre-merge shots into one continuous English prompt (Veo doesn't multi-shot anyway)
- **Only Grok** — aggressively shorten, strip cinematic vocabulary
- **Only HappyHorse** — front-load element definitions, allocate 4 angles even if asset list has fewer

---

## Rhythm analysis (v3.1 core)

The biggest change from v3.0 — and the differentiator of this skill — is that shot count is **not** derived from time. A 6-second meditation and a 6-second sword fight need radically different shot structures.

### Four dimensions

For every row, evaluate:

| Dimension | What to look for | Effect |
|---|---|---|
| **Action density** | Count explosive verbs (炸/劈/扑/碎) vs static descriptions | Extreme → 0.8–1.5s dense cuts; Low → 3–5s long takes |
| **Emotional arc** | Calm / tension / climax / turning point | Maps to rhythm curve (slow / build / fast / freeze) |
| **Dialogue presence** | Has spoken lines? | Each line gets ≥ 2s dwell time |
| **Information density** | Distinct visual points in the sentence | Each major point gets at least one shot |

### Hard constraints (always enforced)

- Each shot **≥ 0.8s** (below this is imperceptible)
- Each shot **≤ 5s** (8s for special long takes)
- ∑ shot durations **= `time` exactly** — no drift
- No gaps between `△` fragments

### Example: same duration, different cuts

```
6 seconds, "我想……我们或许该回去了。"
  → 1–2 long shots:  △[0-3s] speaking shot  △[3-6s] reaction

6 seconds, "他猛地侧身闪开，反手一刀劈下，敌人胸口炸开血花。"
  → 5–6 dense shots:  dodge | regrip | swing | contact | blood mist | aftermath
```

Full decision rules + 5 worked examples: [`references/rhythm-rules.md`](references/rhythm-rules.md).

---

## Shot library

60+ camera shots organized into 9 categories, each with a code (e.g. `A01`, `E09`, `G07`):

| Category | Theme | Example codes |
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

Full library + emotion-to-shot lookup table: [`references/shot-library.md`](references/shot-library.md).

---

## Worked examples

### Example A — Full five platforms

**Input:**

```json
{
  "tableData": [
    { "time": 4, "attribute": "人物·李雷", "originalText": "他长叹一声，转身离去。" }
  ],
  "assetLists": {
    "characters": [
      {
        "name": "李雷",
        "description": "黑发青年，现代休闲装",
        "multi_angle_urls": ["lilei_1.jpg", "lilei_2.jpg", "lilei_3.jpg", "lilei_4.jpg"]
      }
    ]
  }
}
```

**Output (abbreviated):**

```json
{
  "time": 4,
  "attribute": "人物·李雷",
  "originalText": "他长叹一声，转身离去。",
  "sceneDesc": "△[0-2s]近景，李雷@图片2 肩膀微微上提随后缓缓塌落，气息从鼻腔呼出，眼皮沉重半闭 △[2-4s]中景，李雷背影转身向远方走去，散射冷光低饱和",
  "imageRefs": ["李雷"],
  "veoPrompt":        { "prompt": "Medium close-up of a young man, shoulders rising slightly then dropping with a long exhale... Audio: long exhale, distant footsteps.", "reference_images": ["lilei_1.jpg"], "negative_prompt": "..." },
  "grokPrompt":       { "prompt": "@image1 sighs deeply with shoulders dropping, then turns and walks away, melancholic mood, cool desaturated lighting.", "image_urls": ["lilei_1.jpg"], "duration": 4 },
  "seedancePrompt":   { "prompt": "Scene 1: Medium close-up of [Image1]... lens switch. Scene 2: Medium shot, [Image1] turns and walks away. Audio: long exhale, distant footsteps.", "references": [{ "type": "image", "file": "lilei_1.jpg", "index": 1 }] },
  "happyhorsePrompt": { "prompt": "@li_lei sighs heavily with shoulders rising and dropping...", "elements": [{ "name": "li_lei", "description": "young Asian male in casual modern clothes", "element_input_urls": ["lilei_1.jpg", "lilei_2.jpg"] }] },
  "klingPrompt":      { "prompt": "@李雷 肩膀微微上提随后缓缓塌落，气息从鼻腔呼出长叹一声，眼皮沉重半闭，缓慢转身向远方走去。散射冷光低饱和，写实电影质感，真实重力感。", "elements": [{ "name": "李雷", "description": "黑发青年，现代休闲装", "element_input_urls": ["lilei_1.jpg", "lilei_2.jpg", "lilei_3.jpg", "lilei_4.jpg"] }] },
  "prompt": ""
}
```

Notice the **same scene** produces 5 syntactically different prompts:
- Veo: no `@` marker — image is in `reference_images`
- Grok: `@image1`
- Seedance: `[Image1]`
- HappyHorse: `@li_lei` (pinyin)
- Kling: `@李雷` (Chinese)

### Example B — Subset (only Kling + Seedance)

**Request:** *"只生成 Kling 和 Seedance 的"*

**Output:** the same row, but `veoPrompt`, `grokPrompt`, `happyhorsePrompt` are **completely absent** from the output — not `null`, not `{"skip": true}`, just omitted. This keeps the JSONL clean for downstream parsers.

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
- For sequences > 10s, use frame-chain (last frame of clip N → input of clip N+1)

### Seedance 2.0

- Use `lens switch` between scenes (single generation, multi-shot)
- Up to 9 images + 3 videos + 3 audio refs
- Chinese slightly outperforms English (ByteDance origin)
- Background drift in long takes is a known limitation

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
- For 4+ shots, also produce `multi_prompt` array (up to 6 shots)

Full per-platform translation guide: [`references/platform-adaptations.md`](references/platform-adaptations.md).

---

## Common pitfalls

1. **Forgetting `Audio:`** in Veo / Seedance prompts — both have native audio
2. **Using Chinese in HappyHorse element names** — stick to pinyin or English
3. **Forgetting `multi_prompt` for Kling action sequences** with 4+ shots
4. **Mismatched time totals** — always verify ∑(shot durations) === `time`
5. **Falling back to `N = round(time/1.5)`** — defeats the v3.1 differentiator
6. **Repeating `@图片1` for scenes** — scene appears only in the first `△` fragment
7. **Including scene names in `imageRefs`** — only characters and items
8. **Abstract emotion words** — never `"悲伤"` or `"happy"`; physicalize to muscle / posture
9. **Writing `null` for unselected platforms** — just omit the field entirely
10. **Generating all five when user asked for a subset** — re-read the request per row
11. **Mis-resolving aliases** — `可灵` → `kling`, `即梦` → `seedance`, `快马` → `happyhorse`

---

## FAQ

**Q: My input is plain narrative text, not structured JSON. Can the skill still help?**
A: Yes — the skill will first help you format raw text into the `tableData` schema, then proceed. Or you can pre-format using the spec in [`references/input-schema.md`](references/input-schema.md).

**Q: What if my character isn't in `assetLists`?**
A: The character is described by name only (no `@` reference). All platform prompts fall back to text description. The skill flags `"_no_ref": ["X"]` so you know.

**Q: How is `time` enforced? What if I want exactly 6 seconds?**
A: The skill verifies ∑(shot durations) === `time` exactly. Each shot is ≥ 0.8s and ≤ 5s. If the requested duration can't fit the rhythm, the skill warns.

**Q: Can I use this without Claude?**
A: The skill is built for Claude (via `SKILL.md` description triggering). The reference docs are usable as a manual prompt-engineering guide too, but the auto-trigger flow needs Claude.

**Q: Does the skill fetch reference image URLs?**
A: No. URL reachability is a downstream concern — the skill just passes URLs through. Use any image host (your own CDN, S3, etc.).

**Q: How do I update to a new version?**
A: `cd ~/.claude/skills/video-prompt-skill && git pull`.

**Q: Can I add my own platform?**
A: Fork, add a new section to `references/platform-adaptations.md`, add the platform ID to the canonical list in `SKILL.md`, update the output schema. PR welcome.

---

## Project structure

```
video-prompt-skill/
├── SKILL.md                          # Skill entry point — schema, workflow, pitfalls
├── README.md                         # This file (English)
├── README.zh-CN.md                   # 中文版
├── LICENSE                           # MIT
└── references/
    ├── input-schema.md               # Input format + validation
    ├── rhythm-rules.md               # 4-dimension rhythm analysis
    ├── shot-library.md               # 60+ camera shots (A01–I12) + emotion lookup
    ├── platform-adaptations.md       # Per-platform translation guide
    └── full-prompt-v3.1.md           # Full v3.1 system prompt archive
```

References are loaded **on demand** by Claude — not all upfront. This keeps context lean.

---

## What's new in v3.1

| Area | v3.0 | v3.1 |
|---|---|---|
| Shot count | Fixed formula `N = round(time / 1.5)` | **Semantic 4-dimension analysis** |
| Platform selection | All five always | **Subset via NL or JSON field** |
| Output for skipped platforms | (n/a) | **Completely omitted** (no nulls) |
| Single-platform mode | (n/a) | **Per-platform optimizations** |
| Rhythm constraints | Implicit | **Explicit: ≥ 0.8s, ≤ 5s, sum = time** |
| Shot library | 40 shots | **60+ shots, A01–I12** |
| Physics keywords | (n/a) | **Kling-specific keyword list** |

---

## Contributing

Issues and PRs welcome.

Good first PR ideas:
- Add a new shot to `references/shot-library.md` (with code + emotion mapping)
- Document a new platform behavior in `references/platform-adaptations.md`
- Add a worked example for an edge case
- Translate `references/*.md` to another language

For substantial changes (e.g. new platform support, rhythm algorithm changes), open an issue first.

---

## License

[MIT](LICENSE) — use freely in commercial and personal projects.

---

<div align="center">

**Found this useful?** ⭐ Star the repo so others can find it.

</div>
