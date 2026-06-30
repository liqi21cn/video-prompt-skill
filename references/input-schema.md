# Input Schema Reference

Defines the expected input format for the video-prompt skill, plus validation rules and edge-case handling.

## Top-level structure

The skill expects JSON input with two required and one optional top-level field:

```json
{
  "platforms": ["kling", "seedance"],    // optional, see "Platform selection" below
  "tableData": [ /* script rows */ ],     // required
  "assetLists": { /* reference assets */ } // required
}
```

If the user provides only one of the required fields (or provides plain text instead of JSON), help them format it before proceeding.

## Platform selection (optional)

The `platforms` field is optional. When present, it MUST be an array of strings drawn from this canonical set (case-insensitive):

| Canonical ID | Common aliases |
|---|---|
| `veo` | Veo, Veo 4, Veo4, Google Veo |
| `grok` | Grok, Grok Imagine, xAI Grok |
| `seedance` | Seedance, Seedance 2.0, 即梦 |
| `happyhorse` | HappyHorse, Happy Horse, 快马 |
| `kling` | Kling, Kling 3.0 Omni, 可灵 |

Validation rules for `platforms`:

1. **Must be an array** (not a string, not null). Wrong: `"platforms": "kling"`. Right: `"platforms": ["kling"]`.
2. **Each element must be a recognized ID or alias.** Unrecognized values trigger a warning to the user (e.g., "I don't recognize 'sora' — did you mean one of: veo, grok, seedance, happyhorse, kling?").
3. **Empty array** = same as missing = default to all five.
4. **Normalize before processing.** "Kling 3.0", "可灵", "kling" all resolve to canonical `kling`.

If the user's natural-language request ALSO specifies platforms (e.g. "只要 Veo"), the JSON `platforms` field wins. Mention this briefly in the preamble line.

---

## tableData schema

An array of script row objects. Each row represents one continuous beat of the drama (typically 2-15 seconds of screen time).

### Row fields

| Field | Type | Required | Notes |
|---|---|---|---|
| `time` | number | yes | Duration in seconds. Integer preferred, one decimal OK (e.g. 6, 6.5). |
| `attribute` | string | yes | Row attribute label. See "Attribute values" below. |
| `originalText` | string | yes | The dialogue line, narration, or system text. Source content. |

### Attribute values

Common values (the skill treats these specially):

| Attribute | Meaning | Rhythm hint |
|---|---|---|
| `人物·<名字>` | Character dialogue or close-focus action | Per-character pacing; pause for delivery |
| `旁白` | Narrator voice-over | Match prose rhythm; visuals serve narration |
| `系统` | System UI / RPG-style alert | Short shots + reaction combo |
| `环境` | Pure environmental description | Free intercutting allowed |
| `字幕` | On-screen text/subtitle beat | Hold long enough to read |

### Unknown attribute fallback

If `attribute` is empty, missing, or doesn't match the patterns above, default to treating the row as `旁白` (narrator) — slow pacing, environmental focus, no character-specific lipsync triggers.

### Example tableData

```json
{
  "tableData": [
    {
      "time": 4,
      "attribute": "环境",
      "originalText": "深夜的修炼密室，烛火轻摇。"
    },
    {
      "time": 6,
      "attribute": "人物·赵长安",
      "originalText": "他盘膝而坐，灵气如潮水般涌入丹田。"
    },
    {
      "time": 3,
      "attribute": "系统",
      "originalText": "【突破提示】距离金丹期还剩 3%。"
    }
  ]
}
```

---

## assetLists schema

A grouped registry of reference assets organized by type. Three top-level groups, each is an array of asset objects:

```json
{
  "assetLists": {
    "scenes":     [ /* scene assets */ ],
    "characters": [ /* character assets */ ],
    "items":      [ /* item / prop assets */ ]
  }
}
```

### Asset object fields

Every asset (regardless of group) has the same shape:

| Field | Type | Required | Notes |
|---|---|---|---|
| `name` | string | yes | Canonical name as referenced in `originalText`. Must match exactly. |
| `description` | string | yes | Short visual description. Used as fallback when refs fail or for platforms that need rich text. |
| `multi_angle_urls` | string[] | yes | 1-4 image URLs. Order: clearest-identity-first (frontal portrait, then side, etc.) |

### multi_angle_urls usage rules

This is the **most platform-sensitive field**. How many URLs to provide depends on which platforms will consume them:

| Platforms targeted | Recommended URL count | Why |
|---|---|---|
| Only Veo / Grok | 1 (frontal) | They use one image per reference; extras are ignored |
| Only Seedance | 1-3 | Seedance accepts up to 9 numbered images total but one per entity is typical |
| Includes HappyHorse | **2-4 multi-angle** | HappyHorse's element system shines with multi-angle |
| Includes Kling | **4 multi-angle** | Kling element system also wants 4 angles |

If targeting all five platforms (the default), supply **4 angles per character**: front, side, three-quarter, back.

For scenes and items, 1-2 URLs usually suffice (they don't need character-grade multi-angle).

### Example assetLists (complete)

```json
{
  "assetLists": {
    "scenes": [
      {
        "name": "修炼密室",
        "description": "幽暗石室内壁刻满符文，中央有打坐蒲团",
        "multi_angle_urls": [
          "https://cdn.example.com/scenes/chamber_wide.jpg",
          "https://cdn.example.com/scenes/chamber_detail.jpg"
        ]
      }
    ],
    "characters": [
      {
        "name": "赵长安",
        "description": "黑发青年，剑眉星目，玄色道袍",
        "multi_angle_urls": [
          "https://cdn.example.com/chars/zhao_front.jpg",
          "https://cdn.example.com/chars/zhao_side.jpg",
          "https://cdn.example.com/chars/zhao_back.jpg",
          "https://cdn.example.com/chars/zhao_three_quarter.jpg"
        ]
      }
    ],
    "items": [
      {
        "name": "灵石",
        "description": "拳头大小的青色晶体，内有金光流动",
        "multi_angle_urls": [
          "https://cdn.example.com/items/lingshi_1.jpg"
        ]
      }
    ]
  }
}
```

---

## Validation rules

Before generating any output, validate the input. If any of these fail, ask the user to fix instead of guessing:

### Critical (must abort)

1. **Top-level fields present** — both `tableData` and `assetLists` exist.
2. **tableData is non-empty array** — at least one row.
3. **Every row has time, attribute, originalText** — none can be missing.
4. **time > 0** — zero or negative durations are invalid.

### Important (warn but proceed)

5. **time ≤ 15** — Beyond 15s, no single platform can render it in one call. Warn the user and proceed, but note in your reasoning that the row may need splitting downstream.
6. **assetLists names referenced in originalText exist** — if `originalText` mentions "赵长安" but no character named 赵长安 is in assetLists, flag it. The entity will appear as text only, no `@图片N` ref.
7. **multi_angle_urls non-empty** — if an asset has zero URLs, the reference becomes text-only.

### Soft (silent fallback)

8. **URL reachability** — Don't try to fetch URLs. Assume they're valid and let the downstream pipeline handle 404s.
9. **Image format** — Don't validate file extensions. Most platforms accept jpg/png/webp.

---

## Edge cases

### Empty assetLists

If `assetLists` is `{}` or all three groups are empty arrays: the skill still produces all 5 platform prompts, but uses text-only descriptions of subjects (no `@image` refs, no `reference_images`, no `elements`). Quality will be lower but output is still valid.

### Single-row input

Common during testing. Process normally and output one JSONL line.

### Mixed-language originalText

If `originalText` contains Chinese and English mixed: preserve the mix in `sceneDesc`, but translate fully to English for veo/grok/happyhorse prompts and fully to Chinese for kling.

### attribute = "人物·X" but X not in characters list

The character is mentioned but has no visual reference. Output:
- `sceneDesc` describes them by name + visual inference from `originalText`
- All platform prompts use text description (no reference attachment)
- Add `"_no_ref": ["X"]` to imageRefs object (optional flag)

### time = 1 or less

Very short rows. Constraint: single shot only (since min shot is 0.8s). Use a single shot covering the full duration.

### Multiple characters in one row's originalText

All referenced characters that exist in `assetLists` get their own `@图片N` marker. The N counter increments left-to-right by first appearance. Example:

```
originalText: "李雷望着韩梅梅，韩梅梅低头不语"
→ sceneDesc: "镜头1[0-3s]: 教室@图片1，李雷@图片2 望着韩梅梅@图片3..."
→ imageRefs: ["李雷", "韩梅梅"]
```

### originalText is dialogue inside quotes

Treat the quoted text as the spoken line; the surrounding text describes action. The dialogue triggers lip-sync formatting in Kling and HappyHorse.

Example:

```
originalText: 赵长安看着他，缓缓开口："你来晚了。"
→ The "你来晚了" part triggers @赵长安 直视镜头，缓缓开口说道："你来晚了。" in klingPrompt
```

---

## Pre-flight checklist

Before generating prompts, mentally confirm:

- [ ] Both top-level fields present and well-formed
- [ ] Every row has all three required fields
- [ ] Every time value is positive
- [ ] Asset references in originalText resolvable against assetLists
- [ ] multi_angle_urls counts match the platform targets
- [ ] No empty originalText strings

If anything fails the critical tier, stop and ask the user. Don't fabricate missing fields.
