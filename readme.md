# 标注规则

# 🎥 精细化人体动作与物理原理标注规范

本项目旨在构建一个高精度的多模态数据集，重点关注 **微观动作** 与动作背后的 **物理力学机制**。

- **一段连贯的 Caption**（语义更强）；
- 在 Caption 内用 **动作锚点标签 `<A1>...</A1>`** 标记需要对齐的原子动作；
- 用结构化 `actions[]` 为每个锚点提供 **时间/帧对齐**，并可选提供 **reason**（不要求覆盖全部动作）。

> **输出语言要求：**最终的 `caption`、`actions[].description` 与 `actions[].reason` 统一为 **英文**。

---

## 1. 数据结构（JSON Schema）

标注文件以视频为单位，参考字段如下：

{
  "video_name": "xxx.mp4",

  // 使用帧标注，避免 fps 差异导致秒数误差， 加载视频实际fps

  "fps": 30,

  // 一段连贯的叙述文本：用 <A1>...</A1> 标出需要对齐的原子动作
  "caption": "A male athlete stands in the warm-up area. <A1>He shrugs his shoulders and takes a deep breath.</A1> <A2>He walks to the barbell.</A2> ...",

  // 对齐表：每个锚点 id 至少要提供起止时间/帧；reason 可选
  "actions": [
    {
      "id": "A1",

      // 使用frame
      "start_frame": 72,
      "end_frame": 113,

      // 可选：建议保存锚点内的动作文本，便于校验（也可以由程序从 caption 中抽取）
      "description": "He shrugs his shoulders and takes a deep breath.",

      // 可选：不要求覆盖所有动作；只在“有解释价值”时填写
      "reason": "Loosen the upper body and regulate breathing to prepare for the upcoming exertion.",

    }
  ]
}

### 1.1 必须满足的约束（强约束）

- `caption` 中出现的每个 `<Ai>...</Ai>` **必须**在 `actions[]` 中存在对应 `id=Ai` 的条目。
- `id` 在单个视频内必须唯一，默认按时间顺序从 `A1` 递增。
- 时间表达 `start_frame/end_frame + fps`

### 1.2 允许的情况（重要：不是错误）

- **动作并行 / 时间重叠：**不同 `actions[]` 记录允许 `start/end` 重叠（例如“左手拿着杯子”和“右手拿着抹布”同时的）。
- **reason 缺省：**可以不写 `reason`（""置空），因为并不是所有动作都有合理的物理/逻辑解释。

---

## 2. Caption 标注规范（连贯叙述）

**核心目标：语义连贯 + 动作细节真实。**

- **主语统一：**操作者统称为 **“记录者 / the recorder”**。
- **左右区分：**明确 **左手/右手、左/右侧**。
- **手部精细度：**避免笼统的 “take/grab”；描述手指形态、掌向、腕部旋转等。
- **接触几何：**描述接触点与物体部位（handle/rim/edge/inner side 等）。

**参考句式：**

> `[body part/effector] + [hand shape/orientation] + [contact/force verb] + [object + object part]`

---

## 3. Action Anchor（`<A1>`...`</A1>`）标注规范

### 3.1 什么时候要打锚点？

为了动作的“可对齐/可检索/可解释”，要求把每个微小细节都打锚点。（尤其是：**接触发生、抓取/释放、开合、倒/倒出、按压、旋拧、放置、关键姿态调整、关键受力动作**）。

### 3.2 锚点文本怎么写？

- 锚点内尽量写 **一个主动作**（可包含必要的形态细节），避免把多个动作塞进同一个 `<Ai>`。
- 若存在 **同一主动作的多个阶段**（例如两次短暂握持），仍需要拆分开，而不是让一个锚点出现多个分散时间窗。

### 3.3 并行动作怎么处理？

- 拆成多个锚点，并允许不同锚点的 `start/end` 有重叠。

---

## 4. Reason 标注规范（可选解释）

**核心原则：物理优先 (Physics First)，逻辑为辅。**
解释“为什么要这样操作”时，优先从力学角度分析，其次是任务逻辑。

### 4.1 物理力学维度（High Priority）

- **摩擦力 (Friction)：**解释抓握方式（增加接触面积、防滑落）。
- **力矩与杠杆 (Torque/Leverage)：**开门、拧盖、按压手柄（克服铰链阻尼、利用杠杆放大力矩）。
- **重力 (Gravity)：**倒水、排空、垂手（利用重力排空液体、势能转化）。
- **螺旋/机械结构：**旋转方向、密封圈压缩等。

### 4.2 任务逻辑维度（Secondary）

- **预备：**为下一步腾出空间/调整姿态。
- **安全：**激活安全互锁/避免碰撞等。

> **重要说明：**Reason 不要求覆盖全部锚点；没有合理 reason 的动作可以让""留空。

---

## 5. ✅ 标注示例

### 示例 1：举重准备动作（Caption + Anchors）

```text
A male athlete stands in the preparation area. <A1>He shrugs his shoulders and takes a deep breath.</A1>
<A2>He walks to the barbell.</A2> <A3>He adjusts his feet to shoulder width.</A3>
<A4>He raises his head, fixes his gaze forward, and inhales deeply.</A4>
```

```json
{
  "video_name": "xxx.mp4",
  "fps": 30,
  "caption": "A male athlete stands in the preparation area. <A1>He shrugs his shoulders and takes a deep breath.</A1> <A2>He walks to the barbell.</A2> <A3>He adjusts his feet to shoulder width.</A3> <A4>He raises his head, fixes his gaze forward, and inhales deeply.</A4>",
  "actions": [
    {
      "id": "A1",
      "start_frame": 72,
      "end_frame": 113,
      "description": "He shrugs his shoulders and takes a deep breath.",
      "reason": "Loosen the upper body and regulate breathing to prepare for the upcoming exertion."
    },
    {
      "id": "A2",
      "start_frame": 197,
      "end_frame": 275,
      "description": "He walks to the barbell.",
      "reason": ""
    },
    {
      "id": "A3",
      "start_frame": 275,
      "end_frame": 310,
      "description": "He adjusts his feet to shoulder width.",
      "reason": ""
    },
    {
      "id": "A4",
      "start_frame": 310,
      "end_frame": 367,
      "description": "He raises his head, fixes his gaze forward, and inhales deeply.",
      "reason": "Stabilize posture and attention before initiating the lift."
    }
  ]
}
```

### 示例 2：打开冰箱拿牛奶（Good vs Bad）

| 维度    | ❌ 错误示例 (Too Simple)                | ✅ 正确示例 (Anchors + Detailed + Physics)                                                                                                                                                                                  |
| ------- | --------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Caption | He opens the fridge and takes the milk. | The recorder faces the fridge.`<A1>`He extends his left hand and pulls the fridge handle outward.`</A1>` `<A2>`He hooks the four fingers of his right hand inside the jug handle and lifts the milk jug out.`</A2>` |
| Reason  | Because he wants milk.                  | A1: Pulling outward helps overcome the magnetic seal and hinge resistance. A2: Gripping the handle provides a stable force application point near the center of mass.                                                       |

---

## 6. 🔑 常用术语速查表

- **力学：** friction, torque, leverage, gravity, inertia, damping, reaction force, point of application
- **动作：** pronation/supination, clockwise/counterclockwise, compress, deform, align, suspend
- **目的：** minimize effort, maximize contact area, seal, safety interlock

---

## 7. 🤖 模型预标注参考（Optional Reference）

为了提高标注效率，我们可能会提供模型生成的预标注数据，仅供参考。预标注可能在时间切分、左右判断、细节完整性、reason 的解释上存在问题，需要人工校正。

**建议检查方向：**

1) **时间轴确认：**是否该拆/该合（依据动作语义完整性）。
2) **事实性修正：**左右手/脚、接触点与受力方式是否与画面一致。
3) **reason 物理化：**是否有明确力学依据，避免“看似专业但未发生”的过度解读。
