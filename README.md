# Video Prompt Skill

> 中文版见 [README.zh-CN.md](README.zh-CN.md)

A Claude skill that converts movie/drama scripts into production-ready prompts for five AI video generation platforms in a single pass.

## Supported platforms

| Platform | Language | Reference syntax | Multi-shot |
|---|---|---|---|
| Veo 4 (Google) | English | ordered image array | no |
| Grok Imagine (xAI) | English | `@image1` | frame-chain |
| Seedance 2.0 (ByteDance) | EN / CN | `[Image1]` | `lens switch` |
| HappyHorse 1.0 (Alibaba) | English | `@element_name` (2-4 angles) | shot array |
| Kling 3.0 Omni (Kuaishou) | Chinese | `@中文名` (4+ angles) | `multi_prompt` |

You can request all five (default) or any subset — by JSON field `platforms` or by natural language like "只生成 Kling 和 Seedance 的".

## What's in this repo

```
video-prompt-skill/
├── SKILL.md                          # entry point + output schema
└── references/
    ├── input-schema.md               # input format & validation
    ├── rhythm-rules.md               # semantic rhythm analysis (v3.1's core change)
    ├── shot-library.md               # 60+ camera shots (A01–I12) + emotion mapping
    ├── platform-adaptations.md       # per-platform translation guide
    └── full-prompt-v3.1.md           # full v3.1 system prompt archive
```

## Install

Drop the `video-prompt-skill/` directory into your Claude skills folder:

```bash
# user-level (available across projects)
git clone https://github.com/<your-user>/video-prompt-skill.git ~/.claude/skills/video-prompt-skill

# or project-level
git clone https://github.com/<your-user>/video-prompt-skill.git .claude/skills/video-prompt-skill
```

Then ask Claude something like:

> 帮我把这段剧本转成 Kling 和 Seedance 的提示词：{...tableData + assetLists...}

Claude will auto-trigger the skill based on the description in `SKILL.md`.

## Input format (quick view)

```json
{
  "platforms": ["kling", "seedance"],
  "tableData": [
    { "time": 4, "attribute": "人物·李雷", "originalText": "他长叹一声，转身离去。" }
  ],
  "assetLists": {
    "scenes":     [{ "name": "...", "description": "...", "multi_angle_urls": [...] }],
    "characters": [{ "name": "李雷", "description": "...", "multi_angle_urls": [...] }],
    "items":      [{ "name": "...", "description": "...", "multi_angle_urls": [...] }]
  }
}
```

Output is JSONL — one row per `tableData` entry, containing only the requested platform fields. See [SKILL.md](SKILL.md#output-schema-strict) for the full schema.

## What makes v3.1 different

The previous version used a fixed formula `shots = round(time / 1.5)`. v3.1 replaces it with **semantic rhythm analysis** along four dimensions (action density, emotional arc, dialogue presence, information density). A 6-second meditation and a 6-second sword fight no longer get the same shot count.

See [references/rhythm-rules.md](references/rhythm-rules.md) for worked examples.

## License

MIT — see [LICENSE](LICENSE).
