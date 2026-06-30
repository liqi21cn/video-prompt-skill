# Shot Library Reference

The 60+ camera shots organized by category. Each shot has a unique code (A01, B02, ..., I12). Use codes when discussing shots; this is the single source of truth.

## Selection algorithm

For each shot, pick a shot via this priority cascade:

```
Step 1  Emotion match → candidate set E
Step 2  Action match (within E) → candidate set A
Step 3  Style match (within A) → final shot

Fallback rules:
  If A is empty → use E (emotion wins)
  If E is empty → use defaults [A01 / B01 / A05 / 中景定镜]
                  + mark imageRefs with "_fallback": true

Priority on conflict: emotion > action > style.
```

---

## Category A: Push/Pull movements

| Code | Name | Effect | Prompt phrase | Best for emotions |
|---|---|---|---|---|
| A01 | 缓慢推入 | Focus detail / tension build / intimacy | "slow dolly-in, speed matching emotion" | Realization, intimacy, build-up |
| A02 | 快速推入 | Sudden crisis / shock | "rapid dolly-in to subject, freeze on key moment" | Crisis, alarm |
| A03 | 缓慢拉出 | Loneliness / departure | "slow dolly-out from close-up to wide" | Farewell, melancholy |
| A04 | 快速拉出 | Revelation / scale shift | "rapid dolly-out revealing full scene" | Plot twist, reveal |
| A05 | 微距缓推 | Extreme suspense / prop reveal | "extreme macro slow push toward detail" | Mystery setup |
| A06 | 推进亲密 | Psychological closeness | "smooth dolly toward face, empathy build" | Tenderness, empathy |
| A07 | 后退疏远 | Coldness / closure | "slow recede from subject, distance creating" | Endings, coldness |
| A08 | 希区柯克变焦 | Space distortion / sudden awareness | "dolly-zoom with background stretching" | Epiphany, panic |

## Category B: Movement trajectories

| Code | Name | Effect | Prompt phrase | Best for emotions |
|---|---|---|---|---|
| B01 | 水平横移 | Parallel narrative | "horizontal tracking shot following subject" | Comparison, parallels |
| B02 | 垂直升降 | Vertical layering | "vertical lift or descent revealing scale" | Grandeur, pressure |
| B03 | 对角线移动 | Imbalance / tension | "diagonal camera movement breaking symmetry" | Tension, kinetic |
| B04 | 弧形绕摄 | Hero entry | "semi-circular arc around subject" | Hero moment |
| B05 | 环绕运镜 | Total reveal / spotlight | "360-degree orbit, low to high sweep" | Victory, entry |
| B06 | Z 轴纵深 | Depth immersion | "forward dolly with deep focus" | Immersion, space |
| B07 | 穿梭运动 | Layered atmosphere | "weaving camera through foreground elements" | Period drama, dimensional |

## Category C: Special movement / speed

| Code | Name | Effect | Prompt phrase | Best for emotions |
|---|---|---|---|---|
| C01 | 手持灵动 | Immersion / breath | "handheld with subtle breathing shake" | Documentary, tension |
| C02 | 轨道平移 | Pure stability | "perfectly steady tracking on dolly rail" | Beauty, art-film |
| C03 | 俯冲下降 | Pressure / drop | "high-angle aerial diving downward fast" | Pressure, danger |
| C04 | 爬升仰视 | Heroic ascent | "low-angle upward climb" | Heroic, entry |
| C05 | 渐进加速 | Emotional escalation | "camera movement accelerating gradually" | Build-up |
| C06 | 急停定帧 | Impact / freeze | "rapid move then snap freeze" | Shock, reversal |
| C07 | 旋转运镜 | Vertigo / dream | "360-degree rotation around center" | Dream, vertigo |

## Category D: Composition / angle

| Code | Name | Effect | Prompt phrase | Best for emotions |
|---|---|---|---|---|
| D01 | 荷兰角 | Imbalance / villain | "Dutch angle 15-30 degrees tilt, vignette" | Tension, unease |
| D02 | 上帝视角 | Solitude / fate | "directly overhead bird's-eye view" | Loneliness, doom |
| D03 | POV 第一人称 | Total immersion | "first-person POV, eye-movement simulation" | Exploration |
| D04 | 压迫俯拍 | Dominance | "high-angle looking down at subject" | Pressure, threat |
| D05 | 无力仰角 | Smallness / awe | "low-angle looking up at subject" | Helplessness, awe |
| D06 | 过肩镜头 | Dialogue tension | "over-shoulder, focus on opposite character" | Dialogue, confrontation |
| D07 | 窥视角度 | Secret / spy | "voyeuristic with foreground obstruction" | Suspense, peeking |
| D08 | 分屏对比 | Dual perspective | "split-screen showing two simultaneous views" | Comparison, parallel |

## Category E: Detail / emotion close-ups

| Code | Name | Effect | Prompt phrase | Best for emotions |
|---|---|---|---|---|
| E01 | 眼神特写 | Emotion bomb | "eye close-up, reflection in pupil shows context" | Sadness, resolve |
| E02 | 瞳孔放大 | Shock / awe | "extreme macro on eye, pupil dilating" | Shock, awe |
| E03 | 颤抖特写 | Tension / fear | "extreme close-up on trembling part, high-freq micro shake" | Tension, fear |
| E04 | 指尖细节 | Tension / heart-flutter | "fingertip detail, brief touch then withdraw" | Anxiety, attraction |
| E05 | 光影变化 | Mood transition | "side-face light shifting from warm to shadow" | Turning point |
| E06 | 呼吸细节 | Suppression / tension | "collar rising and falling, freeze on key beat" | Suppression |
| E07 | 脚步细节 | State externalization | "stable footstep, slight tip-tremor at threshold" | Hesitation |
| E08 | 凝视长镜头 | Resolve / contemplation | "completely static long take, locked gaze" | Resolve, reflection |
| E09 | 眼泪滑落 | Empathy release | "macro tracking single tear descending" | Sadness, release |

## Category F: Psychology / consciousness

| Code | Name | Effect | Prompt phrase | Best for emotions |
|---|---|---|---|---|
| F01 | 旋转晕眩 | Confusion / collapse | "vertigo effect, continuous rotation" | Collapse, chaos |
| F02 | 模糊转清 | Awakening / clarity | "focus transition from blur to sharp" | Realization |
| F03 | 清转变糊 | Fainting / fading | "image slowly going out of focus to blur" | Faint, dissociation |
| F04 | 倾斜失衡 | Mental break / loss of control | "dynamic tilt increasing in angle" | Breakdown |
| F05 | 焦点游移 | Distraction / dazed | "focus wandering between fore and back planes" | Confusion, daze |

## Category G: Action / combat

| Code | Name | Effect | Prompt phrase | Best for emotions |
|---|---|---|---|---|
| G01 | 潜行微动 | Stealth atmosphere | "extremely slow stealthy camera follow" | Espionage, sneaking |
| G02 | 汽车追逐 | Speed kinetics | "intense up-down jolts with motion blur" | Chase, action |
| G03 | 屋顶跑酷 | Agility / leaping | "camera height changing with character's jumps" | Parkour, agility |
| G04 | 打斗跟随 | Combat impact | "close-quarters combat tracking, jolting with punches" | Combat |
| G05 | 格挡震动 | Collision force | "single sharp shake on weapon impact" | Combat impact |
| G06 | 剑光划过 | Weapon trajectory | "ultra-fast camera whip following blade arc" | Wuxia combat |
| G07 | 子弹时间 | Classic slo-mo | "subject mid-air frozen, camera orbiting" | Hero moment |
| G08 | 破门而入 | Extreme impact | "violent forward push through breaking door" | Breach, impact |
| G09 | 受伤踉跄 | Stagger state | "unsteady shaky simulating stumble" | Injury |

## Category H: Transitions / editing

| Code | Name | Effect | Prompt phrase | Best for emotions |
|---|---|---|---|---|
| H01 | 甩镜转场 | Sharp cut | "rapid whip with strong motion blur" | Switching |
| H02 | 遮挡转场 | Seamless | "camera moves behind foreground object, hidden cut" | Polished |
| H03 | 匹配剪辑 | Shape match | "matching geometric shapes between two shots" | Clever |
| H04 | 回忆叠化 | Tender transition | "crossfade between two images overlapping" | Memory, tenderness |
| H05 | 碎片剪辑 | Fragmented memory | "rapid montage of flickering fragments" | Memory, chaos |
| H06 | 长镜压抑 | Claustrophobic | "single continuous long take" | Oppression |

## Category I: Effects / styles

| Code | Name | Effect | Prompt phrase | Best for emotions |
|---|---|---|---|---|
| I01 | 鱼眼扭曲 | Exaggerated POV | "strong spherical distortion at edges" | Whimsy |
| I02 | X 光透视 | See-through | "x-ray visualization showing inner structure" | Sci-fi |
| I03 | 时间倒流 | Reverse motion | "motion fully reversed" | Retro, sci-fi |
| I04 | 分身残影 | Speed trails | "translucent afterimages trailing subject" | Speed |
| I05 | 万花筒 | Dream art | "symmetric geometric reflections" | Dream |
| I06 | 水墨晕染 | Chinese ink | "transition into ink-wash diffusion" | Period drama |
| I07 | 老胶片 | Vintage | "coarse grain, scratches, flicker" | Retro |
| I08 | 故障艺术 | Cyberpunk | "chromatic aberration, pixel tears" | Sci-fi cyber |
| I09 | 粒子消散 | Beautiful dissolve | "subject dissolving into glowing particles" | Beautiful exit |
| I10 | 镜面反射 | Subtle reveal | "camera shooting subject's reflection" | Indirect |
| I11 | 魔法释放 | Spell climax | "rapid pull-out from hand to explosion full-shot" | Magic burst |
| I12 | 热成像 | Infrared vision | "thermal imaging view, heat distribution" | Sci-fi tactical |

---

## Emotion → shot code lookup

| Emotion type | Primary keywords | Recommended codes |
|---|---|---|
| Melancholy / loneliness | 分离、落寞、离开、一个人、沉默 | A03, A07, B07, D02, E08, H06 |
| Tension / fear | 危险、追杀、审判、绝境、逃跑 | A02, C01, C06, D01, E03, E06, G02 |
| Romance / dream | 爱情、表白、回忆、温柔、美好 | A06, B04, C07, E04, H04, I05, I09 |
| Joy / triumph | 胜利、觉醒、逆袭、突破 | B05, C04, C05, G07, I11 |
| Suspense / mystery | 秘密、追踪、线索、密室、未知 | A05, D07, F05, H02, I02, I12 |
| Oppression / despair | 压抑、绝望、黑暗 | D02, F01, F03, H06 |
| Anger / explosion | 愤怒、爆发、怒吼 | A04, C06, G04, G05, I11 |
| Sweetness | 甜蜜、依偎、心动 | A06, E04, I05, I09 |
| Resolve / sacrifice | 决绝、牺牲、壮烈 | C04, D05, G07, I11 |
| Memory / time | 回忆、过去、怀念 | F02, H04, H05, I07 |

## Scene-mode combos

| Scene mode | Recommended combo |
|---|---|
| Emotional climax | A01 → A04 → B05 → I11 |
| Tension / chase | D01 + H01 + C06 + G02 |
| Loneliness / farewell | A07 + D02 + H06 |
| Romance / healing | A06 + B04 + I09 + H04 |
| Combat / action | G04 + G05 + G07 + G08 |
| Memory / time passing | H04 + H05 + I07 |
| Sci-fi / magic transform | I11 + B05 + I08 |
| Period / wuxia | B04 + I06 + B07 + I09 |
| Character entry | C04 + B04 + A01 |
| Mental collapse | F01 + F04 + F03 + C07 |
