---
name: video-prompt
description: Convert a movie/drama script (tableData with time, attribute, originalText fields + asset lists with reference images) into production-ready prompts for AI video generation platforms - Veo 4, Grok Imagine, Seedance 2.0, HappyHorse 1.0, and Kling 3.0 Omni. Users can request all five platforms (default) or specify a subset (e.g. "只生成 Kling 的", "just Veo and Seedance"). Use this skill whenever the user provides script data and asks for video generation prompts, storyboards, AI video prompts, multi-shot breakdowns, 分镜, 画面 prompts, or asks to convert text/scripts into video AI prompts. Also use when the user mentions any of these specific platforms by name and wants script-to-prompt conversion, or when they need multi-platform video prompt generation. Handles rhythm-based shot pacing (not fixed formulas), platform-specific reference image syntax adaptation, performance physicalization, and emotion-to-lighting mapping.
---

# Video Prompt v3.1

A skill for converting drama/movie scripts into multi-platform AI video generation prompts. Outputs can be adapted for any subset of **Veo 4, Grok Imagine, Seedance 2.0, HappyHorse 1.0, and Kling 3.0 Omni** — each with its own reference syntax and language preference.

## When to use this skill

Use this skill when the user provides:
- Script data in JSON form (with `tableData` containing `time`, `attribute`, `originalText` fields)
- Asset lists with character/scene/item names and reference images
- Any request to generate video prompts, storyboards, shot breakdowns, or "分镜"

Also use when the user mentions converting scripts/dialogue/narration to AI video, or names any of the five supported platforms (Veo, Grok, Seedance, HappyHorse, Kling).

If the user provides only narrative text without proper script structure, first help them format it into the expected `tableData` schema (see `references/input-schema.md`), then proceed.

## Platform selection (NEW in v3.1)

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

Only the requested platform fields appear in the output JSON. Non-requested platforms are **omitted entirely** (do not write `null` or `{"skip": true}` — just leave them out). This keeps the JSONL clean for downstream parsers.

Example: if user requests `["kling", "seedance"]`, the output for each row contains:
- `time`, `attribute`, `originalText`, `sceneDesc`, `imageRefs` (always)
- `seedancePrompt` (yes)
- `klingPrompt` (yes)
- `prompt: ""` (always — backward compat)
- NO `veoPrompt`, NO `grokPrompt`, NO `happyhorsePrompt` fields at all

### Single-platform optimizations

When only one platform is selected, you may apply platform-specific optimizations that aren't safe in multi-platform mode:

- **Only Kling**: write `sceneDesc` directly in Chinese with Chinese element names; skip the intermediate `@图片N` translation step
- **Only Veo**: pre-merge all shots into a single continuous English prompt (Veo doesn't support multi-shot anyway)
- **Only Grok**: aggressively shorten — strip cinematic vocabulary, keep visual essentials only
- **Only HappyHorse**: front-load element definitions; pre-allocate 4 multi-angle URLs even if assetLists has fewer

These optimizations are optional — output quality improves but isn't strictly required.

## Core workflow

**Step 0 (NEW)**: Determine the selected platforms (see "Platform selection" above). Default = all five if unspecified.

For each row in `tableData`:

1. **Read `originalText` and analyze rhythm** — Determine how many shots (△ fragments) this row needs based on action density, emotional arc, dialogue presence, and information density. Do NOT use a fixed formula like `N = round(time/1.5)`. See `references/rhythm-rules.md` for the four-dimension analysis.

2. **Assign per-shot durations** — Each shot must be ≥ 0.8 seconds; total must equal the row's `time` field. Durations can be uneven (e.g., 6 seconds split as 1+0.8+0.7+0.7+1.3+1.5 for an action sequence).

3. **Build `sceneDesc`** — The director-readable storyboard text with `△[start-end s]` time anchors and `@图片N` reference markers. This is the canonical mid-form that all platform prompts derive from.

4. **Apply shot selection algorithm** — Match emotion → action → style from the shot library. See `references/shot-library.md`.

5. **Translate to the selected platform prompts only** — Convert `@图片N` markers into each requested platform's specific reference syntax. Skip any platform not in the selected list. See `references/platform-adaptations.md`.

6. **Output JSONL** — One JSON object per row, streamed row-by-row. Include only the requested platform fields.

## Output schema (strict)

Fields marked "always" appear in every row. Fields marked "conditional" only appear if the corresponding platform is in the selected list.

```json
{
  "time": <number>,                              // always
  "attribute": "<unchanged from input>",         // always
  "originalText": "<unchanged from input>",      // always
  "sceneDesc": "<△[time]desc1 △[time]desc2...>", // always
  "imageRefs": ["<names>"],                      // always

  "veoPrompt": {                                 // conditional: only if "veo" in platforms
    "prompt": "<English, 5-element structure>",
    "reference_images": ["<urls, max 3, identity-clearest first>"],
    "negative_prompt": "<exclusions>"
  },
  "grokPrompt": {                                // conditional: only if "grok" in platforms
    "prompt": "<English, short, with @image1 refs>",
    "image_urls": ["<urls, max 3>"],
    "duration": <number>
  },
  "seedancePrompt": {                            // conditional: only if "seedance" in platforms
    "prompt": "<English or Chinese, Scene 1/2/3 with [Image1] refs, 'lens switch' between scenes>",
    "references": [{"type": "image", "file": "<url>", "index": 1}]
  },
  "happyhorsePrompt": {                          // conditional: only if "happyhorse" in platforms
    "prompt": "<English, simple, with @element_name refs>",
    "elements": [{
      "name": "<pinyin or english>",
      "description": "<english>",
      "element_input_urls": ["<2-4 multi-angle urls>"]
    }]
  },
  "klingPrompt": {                               // conditional: only if "kling" in platforms
    "prompt": "<Chinese, with @中文名 refs, physics keywords>",
    "elements": [{
      "name": "<chinese>",
      "description": "<chinese>",
      "element_input_urls": ["<multi-angle urls>"]
    }],
    "multi_prompt": [{"prompt": "<...>", "duration": <number>}]
  },

  "prompt": ""                                   // always (backward compatibility)
}
```

The trailing `"prompt": ""` field is mandatory — downstream tools expect it as an empty string.

## Critical platform differences (memorize these)

| Platform | Language | Reference syntax | Imgs per ref | Multi-shot |
|---|---|---|---|---|
| Veo 4 | English | none (ordered array) | 1 | no |
| Grok Imagine | English | `@image1` | 1 | no (frame chain) |
| Seedance 2.0 | EN or CN | `[Image1]` | 1 | `lens switch` |
| HappyHorse 1.0 | English | `@element_name` | **2-4 multi-angle** | shot array |
| Kling 3.0 Omni | Chinese | `@中文名` | **4+ multi-angle** | `multi_prompt` |

**Why language matters:** Kling is a Chinese-native model — Chinese prompts outperform English. Veo / Grok / HappyHorse are English-native. Seedance handles both well but Chinese is slightly better (ByteDance origin).

**Why HappyHorse and Kling are different:** They support **element systems** where one named element bundles multiple reference angles. Use ALL of the `multi_angle_urls` from the asset list for these two. For Veo / Grok / Seedance, use only the first (frontal) URL per entity.

## Rhythm analysis (key change from v3.0)

Do NOT use `N = round(time/1.5)`. Instead, read the `originalText` semantically:

- **Action density**: 5 verbs in one sentence → 5-6 dense shots (each 0.8-1.5s)
- **Static dialogue**: → 1-2 long shots (each 3-5s)
- **Emotional turn**: → slow-quick-freeze rhythm curve
- **Environmental description**: → 2-3 free shots
- **System UI alert**: → short shots, UI + reaction combo

See `references/rhythm-rules.md` for detailed cases.

**Hard constraints (always enforce):**
- Each shot ≥ 0.8 seconds
- Each shot ≤ 5 seconds (8s for special long takes)
- Total duration = `time` field exactly
- Continuous time (no gaps between △ fragments)

## Reference files

Load these as needed; don't load all upfront:

- **`references/rhythm-rules.md`** — Four-dimension rhythm analysis with worked examples. Load when deciding shot count and durations.
- **`references/platform-adaptations.md`** — Detailed prompt templates, dos and don'ts, and translation examples for all five platforms. Load when generating any platform's prompt for the first time.
- **`references/shot-library.md`** — The 60+ camera shot library (A01-I12) with selection algorithm and emotion mapping. Load when uncertain which shot fits an emotion or action.
- **`references/input-schema.md`** — Expected input format and validation rules. Load if the user's input looks malformed.
- **`references/full-prompt-v3.1.md`** — The complete v3.1 prompt document. Load this if the user asks "what is the full system prompt" or wants to inspect/modify the underlying logic.

## Common pitfalls

1. **Forgetting `Audio:` in Veo / Seedance prompts** — Both have native audio. Always add an `Audio:` segment.
2. **Using Chinese in HappyHorse element names** — Stick to pinyin or English for HappyHorse `name` fields (avoids unicode issues in `@` references).
3. **Forgetting `multi_prompt` for Kling action sequences** — When a row has 4+ shots and you're targeting Kling, also produce the `multi_prompt` array (up to 6 shots).
4. **Mismatched time totals** — Always verify ∑(shot durations) === `time` field before outputting.
5. **Skipping rhythm analysis** — Falling back to the v3.0 fixed formula. The rhythm analysis is the v3.1 differentiator.
6. **Repeating `@图片1` for scenes** — Scene gets marked only in the first △ fragment of the row.
7. **Including scene name in `imageRefs`** — `imageRefs` only contains characters and items, never scenes.
8. **Abstract emotion words** — Never write "悲伤" or "happy"; always physicalize to muscle/posture descriptions.
9. **Writing `null` or `{"skip": true}` for unselected platforms** — Just omit the field entirely. Downstream parsers handle missing keys cleanly; explicit nulls cause noise.
10. **Generating all five when the user asked for a subset** — Re-read the request before each row. The platform list applies to every row.
11. **Mis-resolving platform aliases** — "可灵" → `kling`, "即梦" → `seedance`, "快马" → `happyhorse`. Normalize before processing.

## Quick example

**Input:**
```json
{
  "time": 4,
  "attribute": "人物·李雷",
  "originalText": "他长叹一声，转身离去。"
}
```

**Rhythm analysis:** Low action density + melancholy emotion + 2 visual points → 2 shots, slow rhythm.

**Output (abbreviated):**
```json
{
  "time": 4,
  "sceneDesc": "△[0-2s]近景，李雷@图片2 肩膀微微上提随后缓缓塌落，气息从鼻腔呼出，眼皮沉重半闭 △[2-4s]中景，李雷背影转身向远方走去，散射冷光低饱和【约束】...",
  "imageRefs": ["李雷"],
  "veoPrompt": {"prompt": "Medium close-up of a young man, shoulders rising slightly then dropping with a long exhale, heavy eyelids. Then medium shot of his back as he walks away. Cool overcast lighting, low saturation, melancholic atmosphere. Audio: long exhale, distant footsteps.", "reference_images": ["lilei_1.jpg"], "negative_prompt": "..."},
  "grokPrompt": {"prompt": "@image1 sighs deeply with shoulders dropping, then turns and walks away, melancholic mood, cool desaturated lighting.", "image_urls": ["lilei_1.jpg"], "duration": 4},
  "seedancePrompt": {"prompt": "Scene 1: Medium close-up of [Image1], shoulders rising slightly then dropping with a sigh, heavy eyelids. lens switch. Scene 2: Medium shot, [Image1] turns and walks away. Audio: long exhale, distant footsteps.", "references": [{"type": "image", "file": "lilei_1.jpg", "index": 1}]},
  "happyhorsePrompt": {"prompt": "@li_lei sighs heavily with shoulders rising and dropping, eyelids heavy half-closed, then turns and walks away into the distance, cool overcast melancholic atmosphere.", "elements": [{"name": "li_lei", "description": "young Asian male in casual modern clothes", "element_input_urls": ["lilei_1.jpg", "lilei_2.jpg"]}]},
  "klingPrompt": {"prompt": "@李雷 肩膀微微上提随后缓缓塌落，气息从鼻腔呼出长叹一声，眼皮沉重半闭，缓慢转身向远方走去。散射冷光低饱和，写实电影质感，真实重力感。", "elements": [{"name": "李雷", "description": "黑发青年，现代休闲装", "element_input_urls": ["lilei_1.jpg", "lilei_2.jpg"]}]},
  "prompt": ""
}
```

Notice how the SAME scene description produces 5 syntactically different prompts: no `@` in Veo, `@image1` in Grok, `[Image1]` in Seedance, `@li_lei` (pinyin) in HappyHorse, `@李雷` (Chinese) in Kling.

## Quick example with platform subset

**User request:** "只生成 Kling 和 Seedance 的，input 如下..."

**Resolved platforms:** `["seedance", "kling"]`

**Output (same row as above, abbreviated):**
```json
{
  "time": 4,
  "attribute": "人物·李雷",
  "originalText": "他长叹一声，转身离去。",
  "sceneDesc": "△[0-2s]近景，李雷@图片2 肩膀微微上提随后缓缓塌落，气息从鼻腔呼出，眼皮沉重半闭 △[2-4s]中景，李雷背影转身向远方走去【约束】...",
  "imageRefs": ["李雷"],
  "seedancePrompt": {"prompt": "Scene 1: Medium close-up of [Image1], shoulders rising slightly then dropping with a sigh, heavy eyelids. lens switch. Scene 2: Medium shot, [Image1] turns and walks away. Audio: long exhale, distant footsteps.", "references": [{"type": "image", "file": "lilei_1.jpg", "index": 1}]},
  "klingPrompt": {"prompt": "@李雷 肩膀微微上提随后缓缓塌落，气息从鼻腔呼出长叹一声，眼皮沉重半闭，缓慢转身向远方走去。散射冷光低饱和，写实电影质感，真实重力感。", "elements": [{"name": "李雷", "description": "黑发青年，现代休闲装", "element_input_urls": ["lilei_1.jpg", "lilei_2.jpg"]}]},
  "prompt": ""
}
```

Note: `veoPrompt`, `grokPrompt`, `happyhorsePrompt` are **completely absent** — not set to `null`, not set to `{"skip": true}`, just omitted. This is intentional for clean downstream parsing.

## Confirming the platform list (preamble)

Before streaming the JSONL output, briefly state which platforms you'll generate for. One short sentence — no commentary on the script itself. This gives the user a chance to correct misinterpretation. Format:

```
> Generating for: Kling, Seedance.
{...row 1...}
{...row 2...}
```

Or in Chinese if the user wrote Chinese:

```
> 仅生成 Kling、Seedance 的提示词。
{...row 1...}
```

This preamble line is the ONLY commentary allowed; everything after it is pure JSONL.

## Streaming behavior

After the one-line preamble (see above), output JSONL row-by-row as you process. Do not buffer the whole output. Do not wrap in markdown code fences. Do not add commentary between rows or after the JSONL stream.

Each line must be valid standalone JSON (no leading commas, no array brackets).
