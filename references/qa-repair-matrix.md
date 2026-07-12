# QA 与修复矩阵

| 症状 | 先检查 | 首选修复 | 何时重画 |
| --- | --- | --- | --- |
| 绿色/洋红描边 | key 色距、contact sheet | 提高 key threshold，重抽帧 | 角色本身使用 key 邻近色且无法换 key |
| 透明像素残留 RGB | validation JSON | 重新规范透明像素并导出 | 通常不重画 |
| GIF 尺寸跳变 | 源 strip 与抽帧结果 | 源稳定则 `stable-slots` | 源 strip 本身比例不稳 |
| 基线跳动 | 源 strip 槽位 | `stable-slots` | 源姿势位置本身漂移 |
| 帧数错误 | row prompt、layout guide | 只重做该行 | 始终重做该行 |
| 邻帧碎片 | 槽位宽度、组件分组 | 调整抽帧法 | 源 strip 已重叠 |
| 漂浮符号/烟/尘 | 源 strip | 只重做该行，动作改由身体表达 | 必须重做 |
| running 变成脚跑 | 状态语义 | 重做 `running`，改为专注处理动作 | 必须重做 |
| review 新增放大镜/文件 | canonical prop | 重做 `review`，只用眼神、头、爪 | 必须重做 |
| failed 与 idle 相同 | 表情和轮廓 | 重做 `failed` | 必须重做 |
| waiting 与 idle 相同 | 是否有询问姿态 | 重做 `waiting` | 必须重做 |
| 左右方向错误 | facing、frame order | 正确逐帧镜像或重做 | 非对称身份不可镜像 |
| 右移动步态不连贯 | 体积、基线、交替步 | 重做 `running-right`，再派生 left | 源 strip 不合格 |
| idle 几乎静止 | GIF 而非 contact sheet | 重做细微呼吸/眨眼 | 视觉上无变化时 |
| 身份跨行漂移 | base 与 row 对比 | 只重做漂移行并附 canonical base | 多数行漂移时重做 base |
| 应用看不到宠物 | 目录、manifest、缓存 | Settings → Pets → Refresh | 文件无效时重新打包 |

## 修复后必须重跑

任何源 strip、帧或派生行改变后，都要重跑其下游步骤：

```text
extract -> inspect -> compose -> validate -> contact sheet -> previews -> visual QA
```

不要因为修改范围小而跳过最终视觉 QA。

## 阈值原则

- 不要把某个 chroma 阈值写死为所有宠物的默认值。
- 先确认角色颜色与 key 有足够距离。
- 逐级提高并在每级查看 contact sheet。
- 如果高阈值侵蚀角色，换 key 或重生成背景，不要继续提高。

## 方向行原则

- 先把 `running-right` 做对，再处理 `running-left`。
- 镜像必须逐帧完成并保持帧顺序。
- 修复 right 后必须强制重建 left；不能保留旧派生结果。
- 非对称文字、标志、眼罩、单侧配件或固定方向道具默认不可镜像。
