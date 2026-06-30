<div align="center">

# Video Prompt Skill

**一条剧本 → 五个 AI 视频平台的成品提示词，一次生成。**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Claude Skill](https://img.shields.io/badge/Claude-Skill-8a4fff)](https://docs.claude.com/en/docs/claude-code/skills)
[![Version](https://img.shields.io/badge/version-3.2-blue)](#版本历史)
[![English](https://img.shields.io/badge/lang-English-blue)](README.md)

</div>

---

## 30 秒看完

输入一行剧本（时长、角色、台词），输出 **5 份不同语法/语言的提示词**——分别给 Veo 4、Grok Imagine、Seedance 2.0、HappyHorse 1.0、Kling 3.0 Omni 直接用。

它**不**生成视频，只生成提示词；不调 API、不渲染——它在"剧本"和"五个平台"之间架了一座符合各家官方规范的桥。

```
                      ┌────► Veo 4 prompt (英文，无标记)
                      ├────► Grok prompt (英文，@image1)
剧本 (tableData) ────► 中间分镜 ────► Seedance prompt (中文四段式)
+ 素材库 (assetLists)  (六维必填)   ├────► HappyHorse prompt (英文，多角度元素)
                      └────► Kling prompt (中文，物理关键词)
```

```bash
git clone https://github.com/liqi21cn/video-prompt-skill.git \
  ~/.claude/skills/video-prompt-skill
```

然后丢一段结构化的剧本给 Claude，它会自动触发。

---

## ✅ 能做什么

| 能力 | 说明 |
|---|---|
| **一次生成 5 个平台** | Veo 4 / Grok Imagine / Seedance 2.0 / HappyHorse 1.0 / Kling 3.0 Omni，每家都是该平台原生语法 |
| **平台子集化** | 「只生成 Kling 的」「只要 Veo 和 Seedance」「skip Grok」都识别。未选中的平台字段直接省略 |
| **语义节奏分析** | 6 秒静坐 → 1 个长镜，6 秒打斗 → 5 个密镜，不是 `N = time/1.5` 公式 |
| **六维必填镜头** | 每镜强制包含 景别 / 动作 / 微表情 / 光影 / 运镜 / 音效，缺一个会自检报错 |
| **50 条微表情库** | 从「强忍委屈」到「邪魅试探」按场景标签查，直接用比临场编更电影 |
| **60+ 镜头库** | A01-I12 编码，情绪 → 镜头映射表，自动选合适的运镜 |
| **物理关键词（Kling）** | 自动注入「真实重力」「重心转移」「头发惯性」等关键词提升真实感 |
| **官方规范对齐** | Seedance 严格按火山引擎 `byted-ark-seedance-pe v1.0` 四段式 + 七项防崩坏尾 |
| **结构化输出** | 流式 JSONL，一行一个 row，下游可以直接喂解析器 |
| **多角度图片管理** | HappyHorse 和 Kling 自动用 4 角度多图，Veo/Grok 自动只取正面图 |
| **对白唇形触发** | 检测到引号台词时，Kling 自动加 `直视镜头，缓缓开口说道：` 触发唇形系统 |
| **音频段自动补全** | Veo 加 `Audio:` 段，Seedance 写 `环境音 / 台词 / 音效` 段 |

---

## ❌ 不能做什么 / 边界

一目了然，避免使用预期落差：

| 不做的事 | 为什么 / 替代方案 |
|---|---|
| ❌ **不生成视频本身** | 只产 prompt，需要你拿到 Veo/Kling/etc. 平台去渲染 |
| ❌ **不调 API** | 输出是文字，不会自动调用任何平台的 API |
| ❌ **不验证图片 URL** | 假定你给的 URL 可访问，不会去 fetch 检查 |
| ❌ **不能脱离 Claude 自动触发** | 参考文档可以人工读，但自动触发流程需要 Claude Code |
| ❌ **只支持这 5 个平台** | Sora、Runway、Pika、Hailuo、Vidu 未支持（欢迎 PR）|
| ❌ **不会从 idea 写剧本** | 你需要先有剧本——skill 帮你把剧本翻译成 prompt，不帮你创作剧情 |
| ❌ **不接受图片/视频输入** | 输入只能是文字剧本 + 图片 URL 字符串 |
| ❌ **跨行连续性弱** | `tableData` 每一行独立处理，多行之间不会自动保证镜头连续性 |
| ❌ **不做后期** | 字幕、调色、剪辑、配音、合成都不管 |
| ❌ **不优化 negative_prompt 玄学** | 用默认 negative_prompt，不会针对你的题材魔改 |

---

## 🆚 和其他方法对比

| 维度 | **本 skill** | 直接问 ChatGPT/Claude | 平台内置助手（如 Kling 智能扩写）| 手工写 | 单平台模板库 |
|---|---|---|---|---|---|
| 多平台一次出 | ✅ 5 个 | ❌ 要分别问 5 次 | ❌ 只该平台 | ❌ 累 | ❌ 只该平台 |
| 平台规范遵守 | ✅ 强约束（v3.2 自检）| ⚠️ 不知道最新规范 | ✅ 自家规范 | ⚠️ 容易漏 | ⚠️ 模板僵化 |
| 跨平台语义一致 | ✅ 同一中间分镜派生 | ❌ 难保证 | ❌ 不可能 | ⚠️ 看人 | ❌ |
| 镜头节奏判断 | ✅ 语义四维分析 | ⚠️ 通常一镜到底 | ⚠️ 通常一镜到底 | ✅ 看人 | ❌ |
| 微表情/镜头库 | ✅ 50+60 条 | ❌ 临场编 | ❌ 一般没有 | ✅ 看人 | ⚠️ 套话 |
| 物理关键词（Kling）| ✅ 内置 | ❌ 一般不会 | ⚠️ 自家版本 | ⚠️ 需要熟悉 | ❌ |
| Seedance 官方四段式 | ✅ 强制 | ❌ 通常用旧格式 | ✅ | ⚠️ 容易写错 | ⚠️ |
| 结构化 JSONL 输出 | ✅ | ❌ markdown 散文 | ❌ 网页 UI | ❌ | ⚠️ |
| 可定制 / 开源 | ✅ MIT | ❌ | ❌ 黑盒 | ✅ 你写你拥有 | ⚠️ 看 |
| 速度 | ⚡ 单次对话 | 🐢 5 次对话 | 🐢 每平台手动 | 🐢🐢 | ⚡ 但只 1 个平台 |
| 跑批生产 | ✅ 流式 JSONL | ❌ 难 | ❌ 不可能 | ❌ | ⚠️ |

**简单总结：**
- 想试一下 → 平台内置助手（一个平台）
- 偶尔写几条 → 手工 + 查文档
- **批量出活 / 多平台 A/B / 一致风格 → 本 skill**
- 已经有一套自己的工作流 → 拿这里的 [`references/`](references/) 当资料库

---

## 🚀 快速开始

### 1. 装

```bash
# 用户级（跨项目可用）
git clone https://github.com/liqi21cn/video-prompt-skill.git \
  ~/.claude/skills/video-prompt-skill

# 或项目级（仅当前项目）
git clone https://github.com/liqi21cn/video-prompt-skill.git \
  .claude/skills/video-prompt-skill
```

### 2. 验

重启 Claude Code → 问「列出我可用的 skills」→ 应该能看到 `video-prompt`。

### 3. 用

```
帮我把这段剧本转成 Kling 和 Seedance 的提示词：

{
  "platforms": ["kling", "seedance"],
  "tableData": [
    { "time": 4, "attribute": "人物·李雷",
      "originalText": "他长叹一声，转身离去。" }
  ],
  "assetLists": {
    "characters": [{
      "name": "李雷", "description": "黑发青年，现代休闲装",
      "multi_angle_urls": ["url1", "url2", "url3", "url4"]
    }]
  }
}
```

skill 会在你提到 *分镜 / 画面 prompts / video prompt / 即梦 / 可灵 / Veo / Grok* 等关键词时自动触发。

---

## 🎯 工作流程（一行剧本会经过什么）

```
1. 读 originalText，做四维节奏分析
   ├─ 动作密度（爆发性动词几个？）
   ├─ 情绪曲线（平静 / 紧张 / 高潮 / 转折？）
   ├─ 对白存在（有台词？每句留 ≥2s）
   └─ 信息密度（几个独立视觉点？）
       │
       ▼
2. 决定镜头数 + 时长分配
   ├─ 每镜 ≥ 0.8s 且 ≤ 5s
   └─ Σ 时长 = `time` 精确相等
       │
       ▼
3. 写中间分镜 sceneDesc（六维必填）
   镜头N[start-end s]: ①景别 ②动作 ③微表情 ④光影 ⑤运镜 ⑥音效
   （③ 微表情建议从 50 条库里挑）
       │
       ▼
4. 派生 5 个平台 prompt
   ├─ Veo: 英文 + 顺序图数组
   ├─ Grok: 英文 + @image1
   ├─ Seedance: 中文四段式 + @图N+名称 + 防崩坏尾
   ├─ HappyHorse: 英文 + @pinyin + 多角度元素
   └─ Kling: 中文 + @中文名 + 物理关键词
       │
       ▼
5. 输出前自检 8 项
   □ 每镜头是否 镜头N[start-end s]: 格式
   □ 六维是否都有
   □ Σ 时长是否对得上
   □ ……
       │
       ▼
6. 流式 JSONL 输出，一行一个 row
```

---

## 📝 完整示例

**输入（4 秒情感戏）：**

```json
{
  "tableData": [
    { "time": 4, "attribute": "人物·李雷",
      "originalText": "他看到她递过来的旧信物，眼眶突然就湿了。" }
  ],
  "assetLists": {
    "characters": [{ "name": "李雷", "description": "黑发青年，现代休闲装",
                     "multi_angle_urls": ["lilei_1.jpg", "lilei_2.jpg"] }],
    "items":      [{ "name": "旧玉佩", "description": "翠绿色雕花边缘有磨损",
                     "multi_angle_urls": ["jade_1.jpg"] }]
  }
}
```

**skill 内部判断：**
- 节奏 → 低动作密度 + 情感转折 + 2 个视觉点 → 2 个长镜头
- 微表情库 → #46 看到旧物 + #07 失而复得（叠加）

**中间分镜 sceneDesc：**

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

**派生的 Seedance prompt（v3.2 中文四段式）：**

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

**派生的 Kling prompt（中文 + 物理关键词）：**

```
@李雷 看着对方递过来的@旧玉佩，目光落在物件上不动眉眼迅速柔软又泛酸；
随后指腹轻轻摩挲玉佩边缘，眼眶瞬间湿润瞳孔放大嘴角想笑却先发抖鼻尖微红手伸到半空停住。
灯笼昏黄暖光从右侧打亮面部左半边在阴影里，玉佩反射温润绿光。
镜头先缓慢推入聚焦玉佩，随后固定长镜头让情绪自然流出。
写实电影质感，真实重力感，呼吸节奏自然，头发惯性。
```

**派生的 Veo prompt（英文 + Audio: 段）：**

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

**注意**：同一段剧情产生了 5 份语法不同的 prompt——`@图N 名称`、`@image1`、`@li_lei`、`@李雷`、纯描述——但语义/情绪/镜头编排是一致的。

---

## 🛠️ 平台速查

| 平台 | 语言 | 引用语法 | 多镜支持 | 一句话特点 |
|---|---|---|---|---|
| **Veo 4** | 英文 | 顺序数组（无标记） | 否 | 提示词执行力最强，原生音频 |
| **Grok Imagine** | 英文 | `@image1` | 帧链 | 迭代快，社媒病毒感 |
| **Seedance 2.0** | **中文** | `@图N + 名称` | **时间片** | 多镜头能力最强，对齐火山官方规范 |
| **HappyHorse 1.0** | 英文 | `@element_name` | shot 数组 | 单人画质最强，多角度元素 |
| **Kling 3.0 Omni** | **中文** | `@中文名` | `multi_prompt` | 物理模拟最强，动作戏专长 |

详细每个平台的写法陷阱见 [`references/platform-adaptations.md`](references/platform-adaptations.md)。

---

## ⚙️ 高级特性

### v3.2 镜头格式硬约束

每镜必须是 `镜头N[start-end s]:` 格式——阿拉伯数字、半角符号。下面这些都判错：

| ❌ 错的 | 错在哪 |
|---|---|
| `△[0-2s] xxx` | 缺 `镜头N` + 冒号 |
| `0-2秒：xxx` | 缺方括号，用中文 秒 |
| `(0-1.5s) xxx` | 圆括号 |
| `第1段 0-1.5s xxx` | `第N段` 不对 |
| `Shot1[0-1.5s]: xxx` | 用了英文 Shot |
| `镜头一[0-1.5s]: xxx` | 中文数字 |

### 六个必填维度

| # | 维度 | 必备要素 |
|---|---|---|
| ① | **景别** | 远景 / 全景 / 中景 / 近景 / 特写 / 极近特写 |
| ② | **主体动作** | 谁 + 做什么 + 怎么做 |
| ③ | **微表情** | 眼神 / 眉眼 / 嘴角 / 呼吸 / 小动作（建议从 [`micro-expressions.md`](references/micro-expressions.md) 抽）|
| ④ | **光影** | 方向 + 色温 + 强度 + 阴影 |
| ⑤ | **运镜** | 推 / 拉 / 摇 / 移 / 跟 / 转 + 速度 |
| ⑥ | **音效** | 环境音 / 动作音 / 情绪音 |

不适用的维度（如静止画面没有运镜），明确写 `静止 / 无运镜 / 无动作`——绝不默默省略。

### 50 条微表情库（v3.2 新增）

| 大类 | 条目数 | 适用场景举例 |
|---|---|---|
| 委屈/隐忍 | 6 | 被误会、背锅、临别强撑 |
| 紧张/慌乱/心虚 | 5 | 秘密暴露、欲言又止、破防 |
| 心动/重逢/情感 | 7 | 一见钟情、看到旧物、失而复得 |
| 冷淡/克制/距离感 | 4 | 装冷漠、贵族距离感 |
| 怒意/反击/不服 | 4 | 压怒、冷反击、伤口微笑 |
| 权谋/试探/隐忍 | **11** | 政治紧张、假平静、杀意闪现 |
| 崩溃/疯感/迷失 | 3 | 执念崩溃、记忆唤醒 |
| 强者气场/神性/威压 | 4 | 初恋滤镜、宗主、神性悲悯 |
| 强忍/坚守/信念 | 3 | 噩耗、误会化解 |
| 释然/结局/守护 | 3 | 死里逃生、沉默守护、结局平静 |

每条统一公式：**眼神落点 + 眉眼变化 + 嘴角变化 + 呼吸停顿 + 小动作 + 适用场景**。完整列表 [`micro-expressions.md`](references/micro-expressions.md)。

### Seedance 四段式（v3.2 对齐火山官方规范）

```
[① 全局设定]   @图1 名称+描述，@图2 ……。
[② 时间片]     0-X 秒：@图N 在 @图M 做什么，<六维>；……
[③ 音频段]     台词：「…」；音效：…；环境音：…。
[④ 防崩坏尾]   4K 高清，<风格>；面部稳定不变形、五官清晰、
              人体结构正常、动作自然流畅、不僵硬、画面无卡顿、无闪烁。
```

**硬规则：** `@图N` 后必跟名称（不允许裸 `@图1`）；每个时间片只能有一种运镜；防崩坏尾七项全要。

### Kling 物理关键词

| 关键词 | 效果 |
|---|---|
| `真实重力` | 重力符合直觉的下落 |
| `重心转移` | 行走时的真实重心切换 |
| `脚跟先落地` | 自然的脚跟先着地 |
| `布料下垂` | 自然的衣料垂感 |
| `头发惯性` | 转身时的头发惯性 |
| `液体粘度` | 真实的液体粘度 |

### 镜头库（60+ 条，A01-I12）

9 大类：A 推拉 / B 轨迹 / C 速度 / D 角度 / E 特写 / F 心理 / G 动作 / H 转场 / I 特效。情绪 → 镜头编码映射表见 [`references/shot-library.md`](references/shot-library.md)。

---

## 📂 项目结构

```
video-prompt-skill/
├── SKILL.md                          # 入口——schema、流程、坑
├── README.md                         # 英文 README
├── README.zh-CN.md                   # 本文件
├── LICENSE                           # MIT
└── references/                       # ↓ 都是按需加载，不会一次性全读
    ├── input-schema.md               # 输入格式 + 校验
    ├── rhythm-rules.md               # 四维节奏分析（5 个详例）
    ├── shot-library.md               # 60+ 镜头库（A01-I12）+ 情绪映射
    ├── platform-adaptations.md       # 各平台翻译指南（v3.2 Seedance 重写）
    ├── micro-expressions.md          # 🆕 v3.2: 50 条微表情库
    └── full-prompt-v3.2.md           # 完整 v3.2 系统提示词存档
```

---

## ❓ FAQ

<details>
<summary><b>Q: 我没有结构化的 JSON，只有一段散文剧本能用吗？</b></summary>

可以。skill 会先帮你把散文格式化成 `tableData` 结构再处理。你只要提供文字 + 角色/场景的参考图 URL。
</details>

<details>
<summary><b>Q: 我的角色没在 assetLists 里怎么办？</b></summary>

skill 用纯文字描述，没有 `@` 引用。所有平台 fallback 到纯文字。skill 会用 `"_no_ref": ["X"]` 标记给你看。
</details>

<details>
<summary><b>Q: 时长怎么严格？我要正好 6 秒不多不少。</b></summary>

每行 skill 都会验证 Σ(镜头时长) === `time` 精确相等。每镜 ≥ 0.8s 且 ≤ 5s。如果节奏容不下，会警告。
</details>

<details>
<summary><b>Q: 不用 Claude 能用吗？</b></summary>

自动触发流程需要 Claude Code。但 `references/` 里的文档可以当人工提示词工程指南——比如直接抄微表情条目、镜头库编码。
</details>

<details>
<summary><b>Q: 怎么更新到新版本？</b></summary>

```bash
cd ~/.claude/skills/video-prompt-skill && git pull
```
</details>

<details>
<summary><b>Q: 能加我自己的平台吗（比如 Sora）？</b></summary>

Fork 这个仓库 →
1. 在 `references/platform-adaptations.md` 加一段
2. 在 `SKILL.md` 的平台 ID 列表加一行
3. 更新输出 schema 加 `<your>Prompt` 字段
4. 提 PR
</details>

<details>
<summary><b>Q: 我有 v3.1 的下游流水线，期望 △[time] 和 [Image1] 格式，怎么迁移？</b></summary>

v3.2 是 breaking 格式变更。下游解析器需要更新：
- `sceneDesc` 现在是 `镜头N[start-end s]:`，不是 `△[time]`
- `seedancePrompt.prompt` 现在是中文四段式，不是 `[Image1] + Scene N`
- `seedancePrompt.references` 形状从 `{type, file, index}` → `{tag, name, file}`

输出 schema **字段名不变**，只是 `sceneDesc` 和 `seedancePrompt.prompt` 的**内容**变了。
</details>

<details>
<summary><b>Q: skill 会拉图片 URL 检查存在吗？</b></summary>

不会。URL 可达性是下游的事——skill 只透传。用任何图床都行。
</details>

---

## 📜 版本历史

| 维度 | v3.0 | v3.1 | **v3.2** |
|---|---|---|---|
| 镜头数 | `N = round(time / 1.5)` | 语义化四维分析 | 同 v3.1 |
| 平台子集化 | ❌ 固定五个 | ✅ NL 或 JSON | 同 v3.1 |
| 未选平台输出 | — | 完全省略 | 同 v3.1 |
| 镜头格式 | `△[time]` | `△[time]` | **`镜头N[start-end s]:` 严格** |
| 每镜维度 | 自由 | 自由 | **6 个必填** |
| 微表情 | 临场编 | 临场编 | **50 条库** |
| Seedance 格式 | `[Image1]` 英文 | 同 v3.0 | **中文四段式 + `@图N+名称`** |
| 输出前自检 | 隐式 | 隐式 | **8 项 checklist** |
| 镜头库 | 40 条 | 60+ 条 | 同 v3.1 |
| 物理关键词 | — | Kling 专属 | 同 v3.1 |

Release notes: [v3.2.0](https://github.com/liqi21cn/video-prompt-skill/releases/tag/v3.2.0)

---

## 🤝 贡献

欢迎 Issue 和 PR。

适合 first PR：
- 加新镜头到 `references/shot-library.md`
- 加新微表情条目到 `references/micro-expressions.md`
- 翻译 `references/*.md` 到其他语言
- 补充某平台的最新坑到 `references/platform-adaptations.md`

重大改动（新平台、节奏算法、格式约束）请先开 Issue 讨论。

---

## 📄 许可证

[MIT](LICENSE)——商用、个人都自由。

---

<div align="center">

**觉得有用？** ⭐ 给个 Star 让别人也找得到。

[Issue](https://github.com/liqi21cn/video-prompt-skill/issues) · [Releases](https://github.com/liqi21cn/video-prompt-skill/releases) · [English](README.md)

</div>
