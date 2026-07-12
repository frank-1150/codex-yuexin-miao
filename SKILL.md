---
name: codex-pet-production
description: Standardize, accelerate, generate, repair, validate, package, and install animated Codex pets from character art, meme references, mascot concepts, or user briefs. Use when a user asks to create a Codex pet, turn a character into a full pet atlas, diagnose pet sprite problems, template the pet-production workflow, or hand a repeatable pet-production runbook to another AI.
---

# Codex Pet Production

Produce a validated Codex pet package through the curated `hatch-pet` engine. Apply the fast-path gates and repair policy in this skill so failures are caught before all animation rows are generated.

## Required reading and dependencies

1. Read `references/production-workflow.md` before a full run.
2. Read `references/qa-repair-matrix.md` when validation or visual QA fails.
3. Read the installed `imagegen` skill before any visual generation.
4. Locate and read the installed `hatch-pet` skill and its contract. Search these locations in order:
   - `${CODEX_HOME:-$HOME/.codex}/skills/hatch-pet`
   - `${CODEX_HOME:-$HOME/.codex}/vendor_imports/skills/skills/.curated/hatch-pet`
   - `${CODEX_HOME:-$HOME/.codex}/skills/.curated/hatch-pet`
5. If `hatch-pet` is unavailable, do not invent atlas-processing scripts. Ask to install the curated skill or deliver the brief and row assets without claiming the Codex package is complete.

## Fast workflow

1. Normalize the user input with `assets/pet-brief-template.md`.
2. Separate the Unicode display name from the ASCII `pet_id`; use lowercase letters, digits, and hyphens for `pet_id`.
3. Prepare one run directory with the `hatch-pet` preparation script. Prefer `python3`; do not assume `python` exists.
4. Generate the canonical base first.
5. Generate only `idle` and `running-right` next. Treat them as gates:
   - `idle` proves identity and subtle motion.
   - `running-right` proves frame count, scale stability, baseline, and gait.
6. Stop and repair these gate rows before producing the remaining rows if either fails.
7. Mirror `running-right` into `running-left` only when markings and props are mirror-safe. Preserve frame order and use `--force` when replacing an existing derived row.
8. Generate the remaining semantic rows with at most two visual workers active at once: `waving`, `jumping`, `failed`, `waiting`, `running`, and `review`.
9. Copy each selected output into `decoded/` before marking its job complete. Remove the generated original only after the copy exists.
10. Run deterministic extraction, inspection, atlas composition, atlas validation, contact-sheet generation, and GIF preview rendering.
11. Perform visual QA on the contact sheet and every GIF. Deterministic validation alone is insufficient.
12. Repair the smallest failing scope, rerun downstream processing, then package and install.

## Non-negotiable contract

- Atlas: `1536x1872`, 8 columns × 9 rows.
- Cell: `192x208`.
- Format: transparent PNG or WebP.
- Unused cells: fully transparent with zero hidden RGB residue.
- Package: `${CODEX_HOME:-$HOME/.codex}/pets/<pet_id>/pet.json` plus `spritesheet.webp`.
- Exact row order and frame counts: `idle` 6, `running-right` 8, `running-left` 8, `waving` 4, `jumping` 5, `failed` 8, `waiting` 6, `running` 6, `review` 6.

## Generation rules

- Attach the canonical base, original references, and matching layout guide to every row job.
- Demand exact frame count, flat chroma background, separated full-body poses, stable identity, and no guide marks.
- Preserve the face, markings, palette, outline, body proportions, and signature prop across rows.
- Forbid detached punctuation, icons, smoke, dust, speed lines, motion marks, shadows, glows, and loose effects.
- Express action through the pet's pose and silhouette.
- Keep non-directional `running` as active task work, not literal foot-running.
- Keep `review` prop-free unless the prop belongs to the canonical identity.
- Use the worker prompts in `assets/worker-prompts.md` and require workers to return only a source path and one QA sentence.

## Quality gates

Do not package until all are true:

- Geometry and transparency validation pass with no errors.
- The contact sheet contains the same character in every used cell.
- Preview GIFs show correct semantics, direction, cadence, looping, scale, and baseline.
- No chroma fringe or chroma-adjacent opaque pixels are visible.
- `idle` is visibly animated but calm.
- `running-right` and `running-left` keep stable body volume and coherent alternating gait.
- Failed, waiting, running, and review are visually distinct.

## Repair priority

1. Deterministic extraction or chroma cleanup.
2. One bad row.
3. One derived directional row.
4. Canonical base only when identity is broadly wrong.
5. Never regenerate the full atlas for an isolated failure.

Use `references/qa-repair-matrix.md` to choose the repair. After any repair, rerun every downstream deterministic step and final visual QA.

## Delivery

Report:

- installed package path
- pet ID and display name
- validation outcome
- contact sheet and preview directory
- rows regenerated or derived
- any manual refresh needed in Settings → Pets

Keep a portable backup containing `pet.json`, `spritesheet.webp`, the contact sheet, and preview GIFs.
