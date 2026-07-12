# Codex 宠物生产标准流程

## 目录

1. 目标与产物
2. 输入契约
3. 目录与状态机
4. 快速生产流程
5. 并行策略
6. 透明化与抽帧
7. 安装与刷新
8. 本次实践中的问题
9. 性能与成本优化
10. 完成定义

## 1. 目标与产物

把一张角色图、表情包或文字设定转换为 Codex 可加载的动画宠物包：

```text
~/.codex/pets/<pet_id>/
├── pet.json
└── spritesheet.webp
```

同时保留 QA 产物：

```text
qa/
├── contact-sheet.png
├── previews/*.gif
├── review.json
└── run-summary.json
```

最终图集固定为 `1536x1872`，8×9 网格，每格 `192x208`。

## 2. 输入契约

开始前收集或推断以下字段：

| 字段 | 必需 | 规则 |
| --- | --- | --- |
| display_name | 是 | 用户可见名称，可使用中文 |
| pet_id | 是 | 小写 ASCII、数字、连字符；不要直接从纯中文名称生成 |
| description | 是 | 一句话 |
| references | 建议 | 原始角色、表情包、角色表 |
| identity_lock | 是 | 颜色、脸、轮廓、比例、标志性配件 |
| style | 是 | sticker、pixel、plush、clay 等 |
| avoid | 是 | 禁止文字、阴影、漂浮特效等 |
| chroma_key | 自动 | 选择不会出现在角色中的纯色 |

优先用 `assets/pet-brief-template.md` 形成结构化简报。缺少非关键字段时由 AI 推断，不要中断；只有身份设定互相冲突时才询问用户。

## 3. 目录与状态机

每个视觉作业只允许以下状态：

```text
pending -> generated -> copied -> complete
                    \-> rejected -> regenerate
```

只有输出已复制到 `decoded/` 后才能标记 `complete`。生成服务目录中的文件不是项目事实来源。

推荐运行目录：

```text
run/
├── pet_request.json
├── imagegen-jobs.json
├── references/
│   ├── canonical-base.png
│   ├── reference-*.png
│   └── layout-guides/
├── prompts/
├── decoded/
├── frames/
├── final/
└── qa/
```

## 4. 快速生产流程

### Phase A：预检

1. 定位 `imagegen` 与 `hatch-pet` 技能并完整读取。
2. 确认 `python3`、`jq`、Pillow 和技能脚本可用。
3. 检查目标宠物目录是否已存在，避免意外覆盖。
4. 把中文显示名与 ASCII `pet_id` 分离，例如：

```text
display_name=月薪喵
pet_id=yuexin-miao
```

5. 确认参考图可读，并选择安全的 chroma key。

### Phase B：准备运行

调用 Hatch Pet 的 `prepare_pet_run.py`。在 macOS 上优先使用 `python3`：

```bash
python3 <hatch-pet>/scripts/prepare_pet_run.py \
  --pet-name '<显示名>' \
  --pet-id '<ascii-id>' \
  --display-name '<显示名>' \
  --description '<一句话>' \
  --reference '<绝对路径>' \
  --output-dir '<绝对运行目录>' \
  --pet-notes '<身份锁定>' \
  --style-preset sticker \
  --style-notes '<风格说明>' \
  --force
```

不要手写图集布局或跳过作业清单。

### Phase C：三步闸门

按顺序只生成：

1. `base`
2. `idle`
3. `running-right`

这是最重要的加速策略。主形象不稳定时，后续八行都会浪费；右移动不稳定时，左右两行都会返工。

闸门检查：

- base：单角色、全身、纯色背景、身份正确。
- idle：6 帧，安静但不是静态复制。
- running-right：8 帧，朝右、体积稳定、头身比稳定、基线稳定、步态交替。

任何闸门失败都先修复，不启动剩余动作。

### Phase D：方向派生

只有以下条件都满足时才能镜像：

- 左右脸部花纹对称或镜像仍合理。
- 配件没有固定语义方向。
- 文字、徽标、数字不存在。
- 右移动步态已经通过动效检查。

使用逐帧镜像脚本，保持帧顺序。目标文件已存在时显式使用 `--force`。

### Phase E：最多双路并行

闸门通过后，用最多两个视觉工作者并行生成：

```text
waving   jumping
failed   waiting
running  review
```

每个工作者只做一行，必须读取该行 prompt 并附加全部输入图。返回格式固定为：

```text
selected_source=/absolute/path/output.png
qa_note=<一句话>
```

父代理负责复制、更新清单、抽帧、验证、修复、安装；视觉工作者不得修改清单。

### Phase F：确定性流水线

所有作业完成后依次运行：

1. `extract_strip_frames.py`
2. `inspect_frames.py`
3. `compose_atlas.py`
4. `validate_atlas.py`
5. `make_contact_sheet.py`
6. `render_animation_previews.py`

确定性验证通过后，仍必须查看 contact sheet 和所有 GIF。

## 5. 并行策略

推荐最大并发数为 2 个图像作业，原因：

- 降低身份漂移发生后大面积浪费的概率。
- 便于父代理及时复制和清理生成文件。
- 视觉生成通常比确定性处理更慢，双路已能隐藏大部分等待时间。
- 最终 QA 必须单路进行，避免多个审查结论不一致。

不要并行生成 base、idle 和 running-right；它们有依赖关系。

## 6. 透明化与抽帧

### 绿色描边

默认色距阈值可能保留抗锯齿绿边。处理顺序：

1. 确认绿边不是角色的设计颜色。
2. 提高 chroma key 阈值并重抽帧。
3. 每次调高后重新检查蓝色、黄色等接近 key 的角色区域是否被误删。
4. 重新生成 contact sheet，不要只信 JSON。

本次 sticker 角色在 `#00FF00` 背景上使用阈值 `220` 清除了绿边，且未损伤角色；这不是所有角色的通用常数。建议从 120、160、200 逐级测试，只有角色颜色距离足够大时才使用 220。

### 尺寸跳变

如果源 strip 的比例与基线稳定，但抽帧后 GIF 跳动：

1. 使用 `stable-slots` 重新抽帧。
2. 使用 `--allow-stable-slots` 检查。
3. 重跑图集、验证、contact sheet 和 GIF。

如果源 strip 本身比例变化大，`stable-slots` 无法修复；必须重做该行。

## 7. 安装与刷新

清单格式：

```json
{
  "id": "pet-id",
  "displayName": "宠物名称",
  "description": "一句话。",
  "spritesheetPath": "spritesheet.webp"
}
```

写入：

```text
${CODEX_HOME:-$HOME/.codex}/pets/<pet_id>/
```

应用已经打开时，自定义宠物查询可能被缓存。告诉用户进入 Settings → Pets 点击 Refresh，然后 Select；不要通过修改内部全局状态文件强制选择。

## 8. 本次实践中的问题

### 问题 1：纯中文名称无法生成合法目录 ID

症状：`pet id must contain at least one letter or digit`。

处理：保留中文 `displayName`，显式传入 ASCII `pet_id`。

预防：预检阶段始终独立生成 ID。

### 问题 2：系统只有 `python3`

症状：`python: command not found`。

处理：改用 `python3`。

预防：运行前执行 `command -v python3`，不要假设 `python` 别名存在。

### 问题 3：失败动作产生漂浮符号和烟气

原因：模型用装饰性特效表达失败，但透明精灵要求效果必须与主体连接。

处理：只重做 failed 行，要求通过低垂耳朵、蜷缩身体和表情表达。

预防：每个动作 prompt 明确禁止漂浮符号、烟气、阴影和分离特效。

### 问题 4：所有帧出现绿色边缘

原因：初始 chroma 阈值过低，抗锯齿像素残留。

处理：确认角色颜色安全后，提高阈值并重跑确定性流水线。

预防：在完成全部动作前，用 base、idle、running-right 做一次透明化测试。

### 问题 5：左右移动比例跳变和步态不连续

原因：第一次右移动源 strip 的体积、头身比和姿势幅度变化过大；抽帧又进一步放大差异。

处理：先测试 `stable-slots`；仍失败后只重做 running-right，要求同体积、同基线、交替步态，再强制重建 running-left。

预防：把 rightward gait 设为三步闸门之一。

### 问题 6：镜像脚本没有覆盖旧文件

症状：`running-left.png already exists; pass --force to replace it`。

处理：修复右移动后，重新派生左移动时使用 `--force`。

### 问题 7：应用不立即显示新宠物

原因：自定义宠物查询使用缓存。

处理：Settings → Pets → Refresh。

## 9. 性能与成本优化

1. 先闸门、后并行，不一次生成全部动作。
2. 用角色表作为参考，但另建 canonical base，避免每行从多角色画面自行猜身份。
3. 每个工作者只生成一个视觉作业，输出只返回路径和 QA 结论。
4. 使用右移动派生左移动，前提是镜像安全。
5. 失败时只修复最小范围。
6. 先修抽帧和透明化，再重画；很多问题是确定性后处理造成的。
7. 生成文件复制成功后立即清理生成缓存，减少存储与上下文负担。
8. 最终只保留 pet request、WebP、验证 JSON、contact sheet、GIF 和 run summary。
9. 将输入模板、worker prompt 和 QA 矩阵固定化，避免每次重新设计提示词。

## 10. 完成定义

只有满足以下条件才能说“已加入宠物”：

- `pet.json` 与 `spritesheet.webp` 已在目标目录。
- 图集尺寸、格式、透明度、未使用格均合格。
- 确定性检查无错误。
- contact sheet 人工或模型视觉 QA 通过。
- 所有 GIF 的身份、动作、方向、比例、基线和循环通过。
- 用户知道必要时刷新 Pets 列表。
- 已保存可移植备份。
