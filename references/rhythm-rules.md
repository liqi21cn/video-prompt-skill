# Rhythm Analysis Rules

The v3.1 prompt system replaces the rigid `N = round(time / 1.5)` formula with semantic rhythm analysis. This file is the detailed reference.

## Why this changed

A 6-second silent meditation and a 6-second sword fight need radically different shot counts. The old formula gave both 4 shots — wrong for both cases. Analyze the script's actual content instead.

## The four dimensions

For each `originalText`, evaluate all four:

### Dimension 1: Action density

Count the verbs and check whether they're explosive or static.

| Density | Indicators in originalText | Per-shot duration |
|---|---|---|
| **Extreme** | 3+ explosive verbs (炸开/撕裂/扑出/猛然) | 0.8-1.5s, dense cuts |
| **High** | 2-3 sequential actions with direction | 1.5-2.5s |
| **Medium** | Single action or state | 2-3s |
| **Low** | Static, dialogue, inner monologue | 3-5s, long takes |

### Dimension 2: Emotional arc

| Emotion type | Rhythm |
|---|---|
| Calm / introspection / monologue | Slow, long takes, few cuts |
| Tension / suspense / standoff | Medium tempo, multiple shots build tension |
| Explosive / shock / climax | Fast, multi-angle, intercut close-ups |
| Turning point / reversal | Freeze-frame + reaction shot combo |

### Dimension 3: Dialogue presence

| Situation | Rhythm |
|---|---|
| Has dialogue | Leave dwell time per line. One shot per spoken sentence. Each ≥ 2s. |
| Narration only | Match the prose tempo. Sparse prose = slow shots. |
| System UI | Short shots, UI element + reaction combo |
| Pure environmental | Free intercutting allowed |

### Dimension 4: Information density

Count the distinct visual information points in the sentence. Each key visual point needs at least one shot to carry it.

Example: "夜雨打在青石板上，远处巷口的灯笼摇曳" has 2 information points (rain on stones, lantern in distance) → at minimum 2 shots.

## Hard constraints

These override aesthetic preferences:

| Constraint | Value | Why |
|---|---|---|
| Min per-shot duration | **≥ 0.8s** | Below this, viewers cannot perceive the image |
| Max per-shot duration | **≤ 5s** (8s for special long takes) | Beyond this, pace drags |
| Total per row | **= `time` field, exactly** | No drift |
| Min shots per row | **1** | Single-shot rows are valid |
| Max shots per row | **ceil(time / 0.8)** | Physical ceiling from min duration |

## Decision flow

```
1. Read originalText. Identify primary emotion + primary action type.
2. Evaluate all four dimensions. Note the dominant one.
3. Predict approximate shot count.
4. Assign durations (uneven is fine, often more dramatic).
5. Verify: sum(durations) === time. Adjust if off.
6. Verify: every duration >= 0.8. Merge shots if any is too short.
```

## Worked examples

### Example 1: Static dialogue (6 seconds)

```
originalText: "我想……我们或许该回去了。"
Action density: LOW (no verbs of action)
Emotion: calm, hesitant
Dialogue: YES, one full sentence
Information density: 1 (just the line)
→ Decision: 1-2 long shots
→ Plan: 镜头1[0-3s]: speaking shot  镜头2[3-6s]: reaction shot
```

### Example 2: Fast combat (6 seconds)

```
originalText: "他猛地侧身闪开，反手一刀劈下，敌人胸口炸开血花。"
Action density: EXTREME (闪开 / 反手 / 劈下 / 炸开 = 4 explosive verbs)
Emotion: explosive climax
Dialogue: NONE
Information density: 5 key points (dodge, regrip, swing, contact, blood mist)
→ Decision: 5-6 dense shots
→ Plan: 镜头1[0-1s]: dodge  镜头2[1-1.8s]: regrip  镜头3[1.8-2.5s]: swing
        镜头4[2.5-3.2s]: contact  镜头5[3.2-4.5s]: blood mist  镜头6[4.5-6s]: aftermath
```

### Example 3: Environmental atmosphere (6 seconds)

```
originalText: "夜雨打在青石板上，远处巷口的灯笼摇曳。"
Action density: LOW-MEDIUM (rain falling, lantern swaying — ambient motion)
Emotion: atmospheric, lonely
Dialogue: NONE
Information density: 2 (rain on stones, lantern in distance)
→ Decision: 2-3 free intercuts
→ Plan: 镜头1[0-2.5s]: rain close-up  镜头2[2.5-4s]: wide pullback  镜头3[4-6s]: lantern detail
```

### Example 4: Inner awakening (6 seconds)

```
originalText: "他猛然睁眼，瞳孔骤然放大——他终于明白了。"
Action density: MEDIUM (a single rapid action + a frozen state)
Emotion: turning point, revelation
Dialogue: NONE (inner)
Information density: 2 (the opening, the realization)
→ Decision: slow-quick-freeze rhythm curve
→ Plan: 镜头1[0-2.5s]: closed eyes slow breathing  镜头2[2.5-3s]: eyes snap open
        镜头3[3-6s]: dilated pupil freeze frame
```

### Example 5: System alert + character reaction (3 seconds)

```
originalText: "警告：检测到天劫即将降临。"
attribute: "系统"
Action density: HIGH (alert appears + character reacts)
Emotion: shock, alarm
Dialogue: system text
Information density: 2 (UI alert, reaction)
→ Decision: 2 dense shots
→ Plan: 镜头1[0-1.5s]: full-screen UI alert with shake  镜头2[1.5-3s]: character close-up, dilated pupil
```

## When to use long single-shots

Default to a single 4-8 second take when:
- Emotion needs uninterrupted build (crying, monologue)
- Long tracking action (walking, running)
- Critical dialogue delivery
- Building claustrophobia or oppression

## When to use 4+ dense cuts

Default to dense intercutting when:
- Climactic action/chase
- Multi-character interaction (each needs own coverage)
- Information bombardment (UI + character + environment all changing)
- Tempo shift point (especially slow-to-fast transitions)

## Variable pacing within a single row

Don't always split evenly. Variable rhythm is more dramatic:
- Start slow, accelerate (build-up scenes)
- Start fast, slam to a hold (impact scenes)
- Two beats of normal, one elongated hold (revelation)

Example: A 5-second row could be 1+1+1+1+1 (even, mechanical) OR 0.8+0.8+1.4+2 (build then hold, more interesting).

## Final check before output

- [ ] Sum of durations equals `time` field exactly?
- [ ] Every duration ≥ 0.8?
- [ ] No gaps between shots (end of N = start of N+1)?
- [ ] Total shot count justified by content, not by a formula?
- [ ] At least one shot per major information point?
