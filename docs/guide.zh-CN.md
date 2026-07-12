# Codex 宠物生成标准作业指南

这份指南把“角色参考图 → 九状态动画 → 透明精灵图集 → Codex 宠物安装”变成可交给另一个 AI 执行的标准流程。

## 一句话版本

先结构化用户输入；再依次生成主形象、待机和向右移动作为质量闸门；通过后最多双路并行生成其余动作；最后用确定性脚本抽帧、透明化、组装和验证，并通过 contact sheet 与 GIF 做视觉质检，只修复最小失败范围。

## 固定产物契约

- 图集：`1536x1872` WebP 或 PNG，透明背景。
- 网格：8 列 × 9 行，每格 `192x208`。
- 行顺序与帧数：
  1. idle：6
  2. running-right：8
  3. running-left：8
  4. waving：4
  5. jumping：5
  6. failed：8
  7. waiting：6
  8. running：6（任务处理中，不是脚跑）
  9. review：6
- 未使用格必须完全透明。
- 安装包：`~/.codex/pets/<pet_id>/pet.json` 和 `spritesheet.webp`。

## 用户输入模板

```yaml
display_name: "月薪喵"
pet_id: "yuexin-miao"
description: "一只被工作榨干、发薪日瞬间复活的办公室猫咪。"
references:
  - "角色原图绝对路径"
identity_lock:
  silhouette: "矮胖全身猫"
  face: "蓝色竖眼、小波浪嘴"
  palette: "米黄、白、暖棕、钴蓝"
  signature_prop: "蓝色领带"
style: "手绘贴纸"
mirror_safe: true
avoid:
  - "文字、Logo、阴影"
  - "漂浮符号、烟、尘、速度线"
  - "动作标记和引导线"
```

显示名可以使用中文，但 `pet_id` 必须使用小写 ASCII、数字和连字符。

## 标准流程

### 1. 预检

- 找到并读取 `imagegen` 与 `hatch-pet` 技能。
- 检查 `python3`、`jq` 和图像依赖。
- 检查目标宠物目录是否已经存在。
- 选择不会出现在角色中的 chroma key。
- 建立运行目录与作业清单。

### 2. 三步质量闸门

只先生成：

1. canonical base
2. idle
3. running-right

主形象检查身份；idle 检查细微动画；running-right 检查体积、比例、基线、方向和交替步态。任何一个失败都先修复，不生成剩余动作。

### 3. 方向派生

如果角色的花纹和配件左右安全，就从已通过的 running-right 逐帧镜像 running-left，并保持帧顺序。右移动重做后，必须用 `--force` 重建左移动。

### 4. 双路并行

闸门通过后，最多同时运行两个单行动画工作者：

```text
waving + jumping
failed + waiting
running + review
```

每个工作者只能生成一行，必须附带原始参考、canonical base 和该行 layout guide。返回结果只包含图片路径和一句 QA 结论。

### 5. 确定性后处理

依次完成：

```text
extract -> inspect -> compose -> validate -> contact sheet -> GIF previews
```

JSON 验证通过不等于视觉通过；必须查看 contact sheet 和每一行 GIF。

### 6. 修复和安装

修复优先级：

1. 调整透明化或抽帧。
2. 重做一行。
3. 重做右移动并重新派生左移动。
4. 只有普遍身份错误才重做 base。

通过后写入 `~/.codex/pets/<pet_id>/`。若应用未立即显示，进入 Settings → Pets → Refresh。

## 这次遇到的问题与解决办法

| 问题 | 原因 | 解决 | 下次预防 |
| --- | --- | --- | --- |
| 中文名无法成为 pet ID | slug 需要字母或数字 | 显式使用 `yuexin-miao` | 显示名与 ID 分离 |
| `python` 不存在 | macOS 只有 `python3` | 使用 `python3` | 预检命令位置 |
| failed 有漂浮符号和烟 | 模型用特效表达动作 | 只重做 failed，用身体姿态表达 | 所有动作 prompt 禁止分离效果 |
| 所有角色有绿边 | chroma 阈值过低 | 安全确认后提高阈值并重抽帧 | 三步闸门后先测透明化 |
| 左右移动尺寸跳变 | 源 strip 比例变化，抽帧放大差异 | 先试 stable-slots，仍失败则重做 right | 把 right 设为早期闸门 |
| 左移动没有更新 | 派生文件已存在 | 使用 `--force` | 右移动重做后强制派生 |
| 应用未立即显示 | 宠物查询缓存 | Pets 页面 Refresh | 交付说明固定写入刷新步骤 |

## 加速原则

1. 不要一开始生成全部九行动画。
2. 先用 base、idle、running-right 暴露身份、透明度和步态问题。
3. 只有视觉生成并行；父代理串行维护清单和安装。
4. 最大图像并发为 2。
5. 镜像安全时只生成一个方向。
6. 先修确定性处理，再决定是否重画。
7. 只重做最小失败范围。
8. 把 worker prompt、输入 brief、QA 矩阵固定成模板。
9. 生成文件复制成功后及时清理缓存。
10. 保存可移植备份：manifest、WebP、contact sheet 和 GIF。

## 给另一个 AI 的执行指令

```text
使用 codex-pet-production 工作流。

1. 将用户输入规范化为 pet brief，并将 Unicode display_name 与 ASCII pet_id 分开。
2. 使用 hatch-pet 准备运行目录，使用 imagegen 生成视觉素材。
3. 严格按 base -> idle -> running-right 的顺序执行质量闸门。
4. 右移动未通过稳定比例、基线、方向和步态检查前，不要生成其他动作。
5. 镜像安全时逐帧派生 running-left，保持帧顺序。
6. 最多两个工作者并行生成剩余语义行；每个工作者只做一行。
7. 禁止文字、阴影、漂浮符号、烟、尘、速度线、动作线和分离特效。
8. 运行完整确定性流水线，并查看 contact sheet 与所有 GIF。
9. 透明边缘或抽帧跳动先通过阈值和 stable-slots 修复；源图本身不稳才重画。
10. 只修复最小失败范围，修复后重跑全部下游验证。
11. 只有安装目录中同时存在 pet.json 和 spritesheet.webp，且最终视觉 QA 通过，才能报告完成。
```

## 完成检查表

- [ ] 用户 brief 完整，pet ID 合法。
- [ ] canonical base 通过。
- [ ] idle 通过。
- [ ] running-right 通过。
- [ ] running-left 正确生成或派生。
- [ ] 其余六行语义清楚。
- [ ] 图集几何与透明度验证通过。
- [ ] 没有 chroma fringe。
- [ ] GIF 没有非预期尺寸和基线跳变。
- [ ] 方向、步态和循环正确。
- [ ] 宠物已安装并保存备份。
- [ ] 用户知道如何 Refresh 与 Select。
