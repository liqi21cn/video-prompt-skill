<div align="center">

# Video Prompt Skill

**把影视剧本一次性转换成五个 AI 视频平台可直接使用的提示词。**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Claude Skill](https://img.shields.io/badge/Claude-Skill-8a4fff)](https://docs.claude.com/en/docs/claude-code/skills)
[![Version](https://img.shields.io/badge/version-3.1-blue)](#v31-更新点)
[![English](https://img.shields.io/badge/lang-English-blue)](README.md)

</div>

输入一行剧本，输出五个平台的成品提示词。每个平台都用正确的语言（Kling 用中文，Veo / Grok / HappyHorse 用英文）、正确的引用语法、正确的多镜结构、正确的细节粒度。

---

## 目录

- [为什么需要这个 skill](#为什么需要这个-skill)
- [支持的平台](#支持的平台)
- [快速开始](#快速开始)
- [输入格式](#输入格式)
- [输出格式](#输出格式)
- [平台选择](#平台选择)
- [节奏分析（v3.1 核心）](#节奏分析v31-核心)
- [镜头库](#镜头库)
- [完整示例](#完整示例)
- [各平台细节](#各平台细节)
- [常见坑](#常见坑)
- [FAQ](#faq)
- [项目结构](#项目结构)
- [v3.1 更新点](#v31-更新点)
- [贡献](#贡献)
- [许可证](#许可证)

---

## 为什么需要这个 skill

手工为五个 AI 视频平台分别写提示词是个累活，而且容易翻车：

- **Kling** 要中文提示词、`@元素名` 是中文
- **HappyHorse** 元素名要拼音、每个角色要 2-4 张多角度图
- **Seedance** 要用 `[Image1]` 方括号、场景之间要 `lens switch`
- **Grok** 要短小的英文提示词、用 `@image1`
- **Veo** 完全不用文字标记——纯按顺序传图片数组

这个 skill 把一行剧本一次性生成五个版本（或任意子集），保证语义一致。秘诀：先把每行变成一份中文导演分镜（`sceneDesc`，带 `△[时间]` 锚点和 `@图片N` 标记），再翻译到各平台。

**核心特性：**

- ✅ 五个平台全覆盖（Veo 4、Grok Imagine、Seedance 2.0、HappyHorse 1.0、Kling 3.0 Omni）
- ✅ 子集化：JSON 字段 或 自然语言（"只要 Kling"、"skip Grok"）
- ✅ 语义节奏分析——告别 `N = round(time / 1.5)` 公式
- ✅ 60+ 镜头库，情绪 → 镜头编码映射
- ✅ Kling 物理关键词（`真实重力`、`重心转移`、`头发惯性`）
- ✅ Veo / Seedance 原生音频字段处理
- ✅ 多镜编排（Seedance `lens switch`、Kling `multi_prompt`）
- ✅ 流式 JSONL 输出，方便下游解析

---

## 支持的平台

| 平台 | 出品方 | 语言 | 引用语法 | 单实体图数 | 多镜 |
|---|---|---|---|---|---|
| **Veo 4** | Google DeepMind | 英文 | 无（顺序数组）| 1 | 否 |
| **Grok Imagine** | xAI | 英文 | `@image1` | 1 | 帧链 |
| **Seedance 2.0** | 字节跳动 | 中/英 | `[Image1]` | 1 | `lens switch` |
| **HappyHorse 1.0** | 阿里 | 英文 | `@element_name` | **2-4 多角度** | shot 数组 |
| **Kling 3.0 Omni** | 快手 | **中文** | `@中文名` | **4+ 多角度** | `multi_prompt` |

**优势速览：**

- **Veo** — 最强提示词执行力，适合静态 / 商品 / 微表情
- **Grok** — 迭代最快，社媒/病毒风格，支持图内文字
- **Seedance** — 单次生成的多镜头能力最强，原生音频
- **HappyHorse** — 单人物画质最强，原生多语言唇形同步
- **Kling** — 物理模拟最强，动作编排最强，IP 元素复用

---

## 快速开始

### 1. 安装

把 skill 放到 Claude 的 skills 目录：

```bash
# 用户级（跨项目可用）
git clone https://github.com/liqi21cn/video-prompt-skill.git \
  ~/.claude/skills/video-prompt-skill

# 或项目级（仅当前项目）
git clone https://github.com/liqi21cn/video-prompt-skill.git \
  .claude/skills/video-prompt-skill
```

### 2. 验证

重启 Claude Code 后问：

> 列出我可用的 skills

应该能看到 `video-prompt`。

### 3. 使用

带结构化输入向 Claude 发起对话：

> 帮我把这段剧本转成 Kling 和 Seedance 的提示词：
>
> ```json
> { "tableData": [...], "assetLists": {...} }
> ```

Skill 会在你提到 *分镜 / 画面 prompts / video prompt / 即梦 / 可灵 / Veo / Grok* 等关键词时自动触发。

---

## 输入格式

顶层结构：

```json
{
  "platforms": ["kling", "seedance"],     // 可选，默认五个都生成
  "tableData":  [ /* 剧本行 */ ],          // 必填
  "assetLists": { /* 参考素材 */ }         // 必填
}
```

### `tableData[]` — 剧本行

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `time` | number | 是 | 时长（秒）。典型 2-15。|
| `attribute` | string | 是 | `人物·<名>` / `旁白` / `系统` / `环境` / `字幕` |
| `originalText` | string | 是 | 台词 / 旁白 / 系统文字 |

### `assetLists` — 参考素材

三个分组，每组是数组：

```json
{
  "assetLists": {
    "scenes":     [{ "name", "description", "multi_angle_urls": [...] }],
    "characters": [{ "name", "description", "multi_angle_urls": [...] }],
    "items":      [{ "name", "description", "multi_angle_urls": [...] }]
  }
}
```

**`multi_angle_urls` 给几张图的经验：**

| 目标平台 | 推荐图数 |
|---|---|
| 只 Veo / Grok | 1 张正面 |
| 只 Seedance | 1-3 张 |
| 包含 HappyHorse | **2-4 张多角度** |
| 包含 Kling | **4 张多角度**（正 / 侧 / 3/4 / 背）|

完整 schema 与校验规则见 [`references/input-schema.md`](references/input-schema.md)。

---

## 输出格式

流式 **JSONL** —— 每行一个 JSON 对象。

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

**规则：**

- `time` / `attribute` / `originalText` / `sceneDesc` / `imageRefs` / `prompt: ""` 始终存在
- 平台字段**仅在被请求时**出现——未选中的平台**完全省略**（不写 `null`，不写 `{"skip": true}`）
- 一行一个对象——没有数组方括号、没有 markdown 代码块、没有注释

末尾的 `"prompt": ""` 是向后兼容字段，必须保留。

---

## 平台选择

默认五个平台都生成。可以通过两种方式收窄：

### 方法 1 — 自然语言

| 你说 | 解析为 |
|---|---|
| `只生成 Kling 的提示词` | `["kling"]` |
| `只要 Veo 和 Seedance` | `["veo", "seedance"]` |
| `just Veo` | `["veo"]` |
| `skip Grok and Veo` | `["seedance", "happyhorse", "kling"]` |
| `exclude HappyHorse` | `["veo", "grok", "seedance", "kling"]` |

### 方法 2 — JSON 字段

```json
{ "platforms": ["kling", "seedance"], "tableData": [...], "assetLists": {...} }
```

别名（大小写不敏感）：

| 标准 ID | 识别的别名 |
|---|---|
| `veo` | Veo, Veo 4, Google Veo |
| `grok` | Grok, Grok Imagine, xAI Grok |
| `seedance` | Seedance, Seedance 2.0, 即梦 |
| `happyhorse` | HappyHorse, Happy Horse, 快马 |
| `kling` | Kling, Kling 3.0 Omni, 可灵 |

**两种方式同时出现时**，JSON 字段优先。Skill 会在开头一行说明。

### 单平台优化

当只选了一个平台时，会启用一些额外优化：

- **只 Kling** —— `sceneDesc` 直接写中文，跳过 `@图片N` 中间层
- **只 Veo** —— 把多个镜头合并成一条连续的英文提示词（Veo 反正不支持多镜）
- **只 Grok** —— 激进精简，去掉电影词汇，只保留视觉要点
- **只 HappyHorse** —— 元素定义前置，预分配 4 角度即使素材表没那么多

---

## 节奏分析（v3.1 核心）

相比 v3.0 最大的改动，也是这个 skill 的差异化点：**镜头数不再由时长决定**。6 秒的冥想和 6 秒的剑斗，需要完全不同的镜头结构。

### 四个维度

每行都要评估：

| 维度 | 看什么 | 效果 |
|---|---|---|
| **动作密度** | 数爆发性动词（炸/劈/扑/碎）vs 静态描述 | 极高 → 0.8-1.5s 密集切；低 → 3-5s 长镜 |
| **情绪曲线** | 平静 / 紧张 / 高潮 / 转折 | 映射节奏曲线（慢 / 推进 / 快 / 定格）|
| **对白存在** | 有台词？ | 每句台词留 ≥ 2s 表演时间 |
| **信息密度** | 句子里几个独立视觉点 | 每个主要点至少一个镜头 |

### 硬约束（始终遵守）

- 每个镜头 **≥ 0.8s**（再短肉眼看不到）
- 每个镜头 **≤ 5s**（特殊长镜可 8s）
- ∑ 镜头时长 **= `time` 精确相等**——不能漂移
- `△` 片段之间不能有空隙

### 例子：同时长，不同切法

```
6 秒，"我想……我们或许该回去了。"
  → 1-2 个长镜：△[0-3s] 说话镜  △[3-6s] 反应镜

6 秒，"他猛地侧身闪开，反手一刀劈下，敌人胸口炸开血花。"
  → 5-6 个密镜：闪身 | 反握 | 挥刀 | 接触 | 血雾 | 余韵
```

完整决策规则 + 5 个详细例子见 [`references/rhythm-rules.md`](references/rhythm-rules.md)。

---

## 镜头库

60+ 镜头分为 9 个大类，每个有编码（如 `A01`、`E09`、`G07`）：

| 大类 | 主题 | 示例 |
|---|---|---|
| **A** | 推/拉运镜 | `A01` 缓慢推入, `A03` 缓慢拉出, `A08` 希区柯克变焦 |
| **B** | 轨迹运动 | `B04` 弧形绕摄, `B05` 环绕运镜 |
| **C** | 速度/特殊 | `C01` 手持灵动, `C06` 急停定帧 |
| **D** | 角度/构图 | `D01` 荷兰角, `D05` 无力仰角 |
| **E** | 特写/情绪 | `E01` 眼神特写, `E09` 眼泪滑落 |
| **F** | 心理/意识 | `F01` 旋转晕眩, `F02` 模糊转清 |
| **G** | 动作/打斗 | `G07` 子弹时间, `G08` 破门而入 |
| **H** | 转场 | `H04` 回忆叠化, `H06` 长镜压抑 |
| **I** | 特效/风格 | `I06` 水墨晕染, `I11` 魔法释放 |

**选择算法：** 情绪 → 动作 → 风格 级联，冲突时情绪优先。

完整镜头库 + 情绪映射表见 [`references/shot-library.md`](references/shot-library.md)。

---

## 完整示例

### 示例 A —— 五个平台全出

**输入：**

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

**输出（节选）：**

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

注意**同一个分镜**派生出 5 种不同语法的提示词：
- Veo：没有 `@` 标记——图在 `reference_images` 里
- Grok：`@image1`
- Seedance：`[Image1]`
- HappyHorse：`@li_lei`（拼音）
- Kling：`@李雷`（中文）

### 示例 B —— 子集（只要 Kling + Seedance）

**请求：** "只生成 Kling 和 Seedance 的"

**输出：** 同样一行剧本，但是 `veoPrompt`、`grokPrompt`、`happyhorsePrompt` **完全不出现**——不是 `null`、不是 `{"skip": true}`，就是省略。这是为了下游解析干净。

---

## 各平台细节

### Veo 4

- 不要写精确时序（"走 3 秒然后转身"会失败）
- 不要在一条提示词里塞多个故事弧
- **必须带 `Audio:` 段**——否则浪费原生音频能力
- 3 张参考图的上限是硬限制

### Grok Imagine

- **10 秒硬上限**（某些实现 15 秒）
- 提示词**控制在 100 字符以内**效果最好
- 跳过"4K"、"high quality"这类通用形容词——是噪音
- 想要 > 10 秒的连续片段，用帧链（前一片段的最后一帧作为下一片段的输入）

### Seedance 2.0

- 场景之间用 `lens switch` 分隔（单次生成多镜头）
- 最多 9 张图 + 3 个视频 + 3 个音频
- 中文比英文略好（字节出品）
- 长镜头会有背景漂移，是已知限制

### HappyHorse 1.0

- **元素名必须用拼音或英文**——中文在 `@` 引用里可能失败
- 多角度（每个元素 2-4 张图）是它的杀手锏
- description 字段对身份识别影响很大，要认真写
- 多镜上限 5 个

### Kling 3.0 Omni

- **用中文写提示词**效果最好
- 物理关键词显著提升真实感：
  - `真实重力` — 重力符合直觉的下落
  - `重心转移` — 行走时的真实重心切换
  - `脚跟先落地` — 自然的脚跟先着地
  - `布料下垂` — 自然的衣料垂感
  - `头发惯性` — 转身时的头发惯性
  - `液体粘度` — 真实的液体粘度
- 触发对白唇形：`@角色名 直视镜头，缓缓开口说道："..."`
- 4+ 镜头时，要同时生成 `multi_prompt` 数组（最多 6 镜）

完整各平台翻译指南见 [`references/platform-adaptations.md`](references/platform-adaptations.md)。

---

## 常见坑

1. **Veo / Seedance 忘了 `Audio:`** —— 两家都有原生音频
2. **HappyHorse 元素名用了中文** —— 坚持拼音或英文
3. **Kling 多镜动作忘了 `multi_prompt`**（4+ 镜时必须）
4. **时长加起来对不上 `time`** —— 总要核对一遍
5. **退回 `N = round(time/1.5)`** —— 这是 v3.1 的差异化点，别丢掉
6. **场景 `@图片1` 在多个 △ 重复出现** —— 场景只在该行第一个 △ 出现
7. **`imageRefs` 里塞场景名** —— 只有人物和道具
8. **写抽象情绪词** —— 永远不要写 `"悲伤"` 或 `"happy"`；要物理化到肌肉/姿态
9. **没选中的平台写 `null`** —— 整个字段省略
10. **用户要子集你却出了五个** —— 每行都重读一遍请求
11. **别名解析错** —— `可灵` → `kling`、`即梦` → `seedance`、`快马` → `happyhorse`

---

## FAQ

**Q：我输入的是纯文本剧本，不是结构化 JSON，还能用吗？**
A：可以——skill 会先帮你把文本格式化成 `tableData` 结构再处理。或者你按 [`references/input-schema.md`](references/input-schema.md) 自己整好。

**Q：角色不在 `assetLists` 里怎么办？**
A：只用名字描述，没有 `@` 引用。所有平台都退回到纯文字描述。skill 会用 `"_no_ref": ["X"]` 标记给你看。

**Q：`time` 是怎么严格执行的？我想要正好 6 秒。**
A：skill 会验证 ∑(镜头时长) === `time` 精确相等。每个镜头 ≥ 0.8s 且 ≤ 5s。如果时长容不下节奏，会警告。

**Q：不用 Claude 能用吗？**
A：这个 skill 是为 Claude 设计的（通过 `SKILL.md` 描述自动触发）。参考文档也能当人工提示词工程指南用，但自动触发流程需要 Claude。

**Q：skill 会拉取参考图 URL 吗？**
A：不会。URL 可达性是下游的事——skill 只透传 URL。用任何图床都行（自建 CDN、S3 等）。

**Q：怎么更新到新版本？**
A：`cd ~/.claude/skills/video-prompt-skill && git pull`。

**Q：我能加自己的平台吗？**
A：Fork 这个仓库，在 `references/platform-adaptations.md` 加一段，在 `SKILL.md` 的平台 ID 列表里加一条，更新输出 schema，欢迎 PR。

---

## 项目结构

```
video-prompt-skill/
├── SKILL.md                          # Skill 入口——schema、流程、坑
├── README.md                         # 英文 README
├── README.zh-CN.md                   # 本文件
├── LICENSE                           # MIT
└── references/
    ├── input-schema.md               # 输入格式 + 校验
    ├── rhythm-rules.md               # 四维节奏分析
    ├── shot-library.md               # 60+ 镜头（A01–I12）+ 情绪映射
    ├── platform-adaptations.md       # 各平台翻译指南
    └── full-prompt-v3.1.md           # 完整 v3.1 系统提示词存档
```

References 由 Claude **按需加载**——不会一次性全部读入。保持 context 精简。

---

## v3.1 更新点

| 维度 | v3.0 | v3.1 |
|---|---|---|
| 镜头数 | 固定公式 `N = round(time / 1.5)` | **语义化四维分析** |
| 平台选择 | 始终五个 | **NL 或 JSON 字段子集** |
| 未选中平台输出 | （不适用）| **完全省略**（不写 null）|
| 单平台模式 | （不适用）| **专属优化** |
| 节奏约束 | 隐式 | **显式：≥ 0.8s、≤ 5s、sum = time** |
| 镜头库 | 40 个 | **60+ 个，A01–I12** |
| 物理关键词 | （不适用）| **Kling 专属关键词列表** |

---

## 贡献

欢迎 Issue 和 PR。

适合做 first PR 的方向：
- 在 `references/shot-library.md` 加新镜头（带编码 + 情绪映射）
- 在 `references/platform-adaptations.md` 补充新的平台行为
- 加一个边界情况的完整示例
- 把 `references/*.md` 翻译到其他语言

重大改动（如新平台支持、节奏算法变更），请先开 Issue 讨论。

---

## 许可证

[MIT](LICENSE) —— 商用、个人使用都自由。

---

<div align="center">

**觉得有用？** ⭐ 给个 Star，帮别人找到它。

</div>
