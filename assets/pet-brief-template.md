# Codex Pet Brief

```yaml
display_name: "<用户可见名称，可中文>"
pet_id: "<小写 ASCII、数字、连字符>"
description: "<一句话>"

references:
  - path: "<绝对路径>"
    role: "character reference"

identity_lock:
  species_or_form: "<物种或形态>"
  silhouette: "<体型与轮廓>"
  face: "<眼睛、嘴、表情语言>"
  markings: "<固定花纹>"
  palette: "<主要颜色>"
  signature_prop: "<固定配件，没有则 none>"

style:
  preset: "sticker|pixel|plush|clay|flat-vector|3d-toy|painterly|auto"
  notes: "<线条、材质、细节等级>"

personality:
  traits: ["<特征1>", "<特征2>"]
  story_hook: "<一句话角色梗>"

constraints:
  mirror_safe: true
  no_text: true
  no_logo: true
  no_shadow: true
  avoid:
    - "detached effects"
    - "floating punctuation"
    - "speed lines"
    - "guide marks"

delivery:
  install: true
  keep_backup: true
  output_dir: "<绝对路径>"
```

允许 AI 推断 personality、style notes 和 avoid；必须明确 `display_name`、`pet_id`、identity lock 和参考图角色。
