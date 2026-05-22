# Video Prompt Skill

> English version: [README.md](README.md)

一个 Claude skill：把影视剧本一次性转换成五个主流 AI 视频生成平台可直接使用的提示词。

## 支持平台

| 平台 | 语言 | 引用语法 | 多镜支持 |
|---|---|---|---|
| Veo 4（Google） | 英文 | 顺序图片数组（无标记）| 否 |
| Grok Imagine（xAI） | 英文 | `@image1` | 帧链 |
| Seedance 2.0（字节）| 中/英 | `[Image1]` | `lens switch` |
| HappyHorse 1.0（阿里）| 英文 | `@element_name`（2-4 角度）| shot 数组 |
| Kling 3.0 Omni（快手）| 中文 | `@中文名`（4+ 角度）| `multi_prompt` |

默认全出五个平台。也可以子集化——用 JSON `platforms` 字段，或者自然语言「只生成 Kling 和 Seedance 的」。

## 目录结构

```
video-prompt-skill/
├── SKILL.md                          # 入口文档 + 输出 schema
└── references/
    ├── input-schema.md               # 输入格式与校验
    ├── rhythm-rules.md               # 语义化节奏分析（v3.1 核心改动）
    ├── shot-library.md               # 60+ 镜头库（A01–I12）+ 情绪映射
    ├── platform-adaptations.md       # 各平台翻译指南
    └── full-prompt-v3.1.md           # 完整 v3.1 系统提示词存档
```

## 安装

把 `video-prompt-skill/` 目录放到 Claude 的 skills 目录下：

```bash
# 用户级（跨项目可用）
git clone https://github.com/<your-user>/video-prompt-skill.git ~/.claude/skills/video-prompt-skill

# 或项目级
git clone https://github.com/<your-user>/video-prompt-skill.git .claude/skills/video-prompt-skill
```

然后向 Claude 提问，类似：

> 帮我把这段剧本转成 Kling 和 Seedance 的提示词：{...tableData + assetLists...}

Claude 会根据 `SKILL.md` 的描述自动触发这个 skill。

## 输入格式（速览）

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

输出是 JSONL，每行对应 `tableData` 的一行，仅包含被选中的平台字段。完整 schema 见 [SKILL.md](SKILL.md#output-schema-strict)。

## v3.1 的关键改动

旧版本用固定公式 `shots = round(time / 1.5)`。v3.1 改为**语义化节奏分析**，按四个维度判断：动作密度、情绪曲线、对白存在、信息密度。6 秒的静坐冥想和 6 秒的剑斗，不再得到相同的镜头数。

详见 [references/rhythm-rules.md](references/rhythm-rules.md) 中的实例。

## 许可证

MIT — 见 [LICENSE](LICENSE)。
