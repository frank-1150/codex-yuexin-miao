# Worker Prompt Templates

## Base

```text
Generate one hatch-pet base image.

Run dir: <absolute run dir>
Job id: base
Prompt file: <absolute prompt file>
Input images: attach every image listed for this job with its role.

Use imagegen only. Produce one centered full-body pet on the exact flat chroma background. Preserve the brief's identity lock. No text, scenery, shadow, floor, detached effect, or guide mark.

Do not edit manifests, copy outputs, generate rows, process frames, package, or install.
Return exactly:
selected_source=<absolute path>
qa_note=<one sentence>
```

## Row

```text
Generate one hatch-pet row.

Run dir: <absolute run dir>
Row id: <state>
Prompt file: <absolute row prompt>
Retry prompt: <absolute retry prompt>
Input images: attach every image listed for this job with its role.

Use imagegen only. Enforce exact frame count, canonical identity, stable body volume, stable baseline, flat chroma background, separated unclipped full-body poses, and the row semantics. No detached effect, shadow, dust, speed line, punctuation, guide mark, or neighboring-frame overlap. Retry once with the retry prompt only for a transport Bad Request.

Do not edit manifests, copy outputs, process frames, package, or install.
Return exactly:
selected_source=<absolute path>
qa_note=<one sentence>
```

## Final visual QA

```text
Visually QA one finalized Codex pet.

Contact sheet: <absolute path>
Preview directory: <absolute path>
Review JSON: <absolute path>
Validation JSON: <absolute path>

Inspect the contact sheet and every GIF. Verify identity, palette, face, proportions, props, exact row semantics, transparency, scale, baseline, looping, direction, and gait. Fail visible chroma fringe, crop, overlap, detached marks, identity drift, inert idle, wrong direction, size popping, or non-alternating gait.

Do not edit files or package.
Return exactly:
visual_qa=pass|fail
qa_note=<one sentence>
repair_rows=<comma-separated ids or none>
repair_notes=<short row-specific notes or none>
```
