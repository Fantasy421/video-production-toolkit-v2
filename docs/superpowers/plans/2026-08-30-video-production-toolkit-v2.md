# Video Production Toolkit V2 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a lightweight Codex plugin that gives evidence-based video-production recommendations, guides users through four decisions, and delegates batch audio, timing, style, illustration, and Remotion production.

**Architecture:** One director Skill owns user guidance and `workflow.json`. Six focused Skills supply narration production, keyword timing, storyboard recommendations, visual-direction samples, batch illustration, and Remotion preview/export. Normal production uses three media Agents and two analysis/design Agents, while storyboard work remains in the main task.

**Tech Stack:** Codex plugin manifest JSON, Markdown `SKILL.md` entrypoints, lightweight JSON project state, existing media-generation tools selected at runtime, Remotion for editable video production.

**Spec:** `docs/superpowers/specs/2026-08-30-video-production-toolkit-v2-design.md`

## Global Constraints

- Every plugin and Skill description states what evidence it uses, what it recommends or produces, and which user choice it enables.
- Public descriptions never mention historical versions, removed mechanisms, or implementation debt.
- Every user question leads with `推荐`, `依据`, and `可调整项`; the user can accept the recommendation without constructing a plan from scratch.
- Keep exactly four user decision points: script/voice, storyboard/resource scope, visual/motion samples, and video preview.
- The main task owns requirements, copy, user choices, storyboard design, `workflow.json`, and Agent dispatch.
- Normal production dispatches five fresh Agents total: voice, keyword timing, visual style/motion, illustration batch, and Remotion video.
- The three media Agents are voice, illustration, and video; one Agent handles the whole batch for its media type.
- Default shot duration is 3–5 seconds. A shot longer than 5 seconds needs sustained information progress, composition changes, or rich motion.
- A transient production failure gets one retry. A second failure returns immediately with impact and a recommended user choice.
- Production Agents report successes, failures, and paths. The main task uses that report as the next guidance input.
- No `tests/` directory or runtime test suite. Validation is limited to JSON syntax, Skill frontmatter, and plugin structure during repository development.
- Keep every Skill self-contained in one `SKILL.md`; add no references, scripts, registries, schemas, migration layer, or placeholder files unless a concrete implementation need appears.

## Target File Map

```text
.codex-plugin/plugin.json                 Plugin identity, version, positive capability description, Skill discovery path
agents/openai.yaml                        User-facing plugin name and recommendation-centered description
assets/project-template/workflow.json     Minimal resumable project state and pending recommendation
skills/video-director-v2/SKILL.md         User guidance, four decisions, workflow updates, Agent dispatch, retries, targeted redo
skills/voice-producer-v2/SKILL.md         Full narration batch production and real timing report
skills/keyword-timing-v2/SKILL.md         Millisecond keyword/phrase cue extraction from real narration
skills/storyboard-director-v2/SKILL.md    Evidence-based shot, motion-time, illustration, and resource recommendations
skills/visual-style-v2/SKILL.md           Representative visual and motion direction for user approval
skills/illustration-producer-v2/SKILL.md  One-Agent consistent illustration batch production
skills/remotion-video-v2/SKILL.md         Editable Remotion project, captions, motion, preview, and approved export
```

---

### Task 1: Create the plugin shell and resumable project template

**Files:**

- Create: `.codex-plugin/plugin.json`
- Create: `agents/openai.yaml`
- Create: `assets/project-template/workflow.json`

**Interfaces:**

- Consumes: the naming, copy, workflow, and decision constraints in the design spec.
- Produces: plugin id `video-production-toolkit-v2`, Skill discovery at `./skills/`, and the initial `workflow.json` shape consumed by `video-director-v2`.

- [ ] **Step 1: Create the plugin manifest**

Use this exact manifest content:

```json
{
  "id": "video-production-toolkit-v2",
  "name": "video-production-toolkit-v2",
  "version": "0.1.0",
  "description": "Recommend and guide voice-timed video production from script and style choices through batch media, Remotion preview, and export.",
  "skills": "./skills/"
}
```

- [ ] **Step 2: Create the user-facing plugin metadata**

Use this exact content:

```yaml
name: Video Production Toolkit V2
description: Gives evidence-based recommendations and guides users from a video idea to an editable preview and approved export.
```

- [ ] **Step 3: Create the project workflow template**

Use this exact initial shape:

```json
{
  "stage": "needs_brief",
  "confirmed": {
    "script": false,
    "voice": false,
    "storyboard": false,
    "style": false,
    "preview": false
  },
  "choices": {},
  "paths": {
    "script": null,
    "storyboard": null,
    "audio": [],
    "keywordTiming": null,
    "images": [],
    "remotionProject": null,
    "preview": null,
    "export": null
  },
  "pendingDecision": {
    "question": "请提供视频主题、已有文案或目标素材",
    "recommendedOption": "先说明主题、目标受众和发布平台",
    "basis": "这三项足以生成首版脚本、时长和画面建议",
    "adjustable": ["视频时长", "语气", "素材限制"]
  },
  "nextAction": "生成推荐脚本和声音方案"
}
```

- [ ] **Step 4: Verify only the file syntax**

Run:

```bash
python3 -m json.tool .codex-plugin/plugin.json >/dev/null
python3 -m json.tool assets/project-template/workflow.json >/dev/null
```

Expected: both commands exit `0` with no output.

- [ ] **Step 5: Commit the plugin shell**

```bash
git add .codex-plugin/plugin.json agents/openai.yaml assets/project-template/workflow.json
git commit -m "feat: scaffold recommendation-led video toolkit v2"
```

---

### Task 2: Implement the recommendation-led director

**Files:**

- Create: `skills/video-director-v2/SKILL.md`

**Interfaces:**

- Consumes: user brief, all confirmed choices, `workflow.json`, and compact Agent reports containing successes, failures, and paths.
- Produces: approved script/voice choice, approved storyboard/resource choice, approved style/motion choice, preview decision, updated `workflow.json`, and five bounded Agent assignments.

- [ ] **Step 1: Write discriminating frontmatter**

Use:

```yaml
---
name: video-director-v2
description: Recommend and guide knowledge-video decisions from user intent through approved script, voice-timed storyboard, batch media, Remotion preview, and export.
---
```

- [ ] **Step 2: Define the response contract before the workflow**

Add a `## Recommendation format` section requiring every decision prompt to contain:

```markdown
推荐：<one concrete default the user can accept>
依据：<content, audience, platform, timing, asset, or cost facts>
可调整项：<two to four bounded alternatives>
请确认：<one decision only>
```

State that the director may make reasonable creative defaults when evidence is sufficient and must not ask the user to design the process from a blank page.

- [ ] **Step 3: Define the four user decisions exactly**

Add `## Four decisions` with these gates:

1. Script and voice: show recommended script, estimated duration, voice, basis, and adjustable items.
2. Storyboard and resources: show recommended shots, cue times, illustration count/ratio, and resource scope.
3. Visual and motion samples: show the recommended direction and representative samples before batch illustration.
4. Preview: recommend export, local edit, or targeted media redo based on the user's feedback.

Require `workflow.json` to be updated immediately after each confirmation and each Agent result.

- [ ] **Step 4: Define the dispatch sequence and Agent boundaries**

Add `## Dispatch sequence` with this exact dependency order:

```text
decision 1 approved
  -> fresh voice Agent using voice-producer-v2
  -> fresh keyword-timing Agent using keyword-timing-v2 after real audio exists
  -> main task uses storyboard-director-v2
decision 2 approved
  -> fresh visual-style Agent using visual-style-v2
decision 3 approved
  -> fresh illustration Agent using illustration-producer-v2 for the whole batch
audio + keyword timing + required images ready
  -> fresh video Agent using remotion-video-v2
decision 4
  -> approved export or a fresh targeted Agent for the affected media type
```

State explicitly that storyboard design stays in the main task and that one normal project uses three media Agents plus two analysis/design Agents.

- [ ] **Step 5: Define compact production handoffs, retry, and targeted redo**

Require every delegated task to return this semantic shape without introducing a schema file:

```json
{
  "successes": [{"id": "item-id", "path": "project-relative-path", "note": "useful runtime fact"}],
  "failures": [{"id": "item-id", "reason": "actionable cause"}],
  "paths": ["project-relative-path"]
}
```

Add these rules:

- retry one transient failure once;
- after a second failure, recommend retry, tool change, resource supplement, or skip when feasible;
- update progress only for a new artifact, clear failure, or user decision;
- for targeted redo, dispatch a fresh Agent only for the named media and necessary downstream dependency;
- use producer reports as workflow facts and continue guiding the user.

- [ ] **Step 6: Validate the Skill structure**

Run:

```bash
python3 /Users/fantasy/.codex/skills/.system/skill-creator/scripts/quick_validate.py skills/video-director-v2
```

Expected: the validator reports the Skill is valid and no scaffold placeholders remain.

- [ ] **Step 7: Commit the director**

```bash
git add skills/video-director-v2/SKILL.md
git commit -m "feat: add recommendation-led video director"
```

---

### Task 3: Implement full-batch narration production

**Files:**

- Create: `skills/voice-producer-v2/SKILL.md`

**Interfaces:**

- Consumes: approved complete script, selected voice, output directory, and optional pronunciation notes.
- Produces: audio paths, sentence-level start/end/duration values, total duration, and failed sentence ids.

- [ ] **Step 1: Write frontmatter**

```yaml
---
name: voice-producer-v2
description: Generate an approved narration as one consistent batch and report real sentence timing, total duration, successful audio paths, and failures.
---
```

- [ ] **Step 2: Define the batch input and production behavior**

The body must require one fresh Agent to handle the complete approved script with one voice configuration. It may select an available TTS or voice tool appropriate to the request, preserve sentence ids, and keep every generated file inside the project output directory.

Require one automatic retry for transient tool or network failure. After a second failure, keep successful sentences and return the failed ids immediately.

- [ ] **Step 3: Define the timing report**

Require a project-relative JSON timing artifact with this shape:

```json
{
  "voice": "confirmed voice label",
  "audioPath": "audio/voiceover.wav",
  "totalDurationMs": 0,
  "sentences": [
    {"id": "line-001", "text": "approved narration text", "startMs": 0, "endMs": 0, "durationMs": 0}
  ]
}
```

Values must come from the produced audio rather than estimated reading speed. The final message reports successes, failures, audio path, timing path, and total duration.

- [ ] **Step 4: Validate and commit**

```bash
python3 /Users/fantasy/.codex/skills/.system/skill-creator/scripts/quick_validate.py skills/voice-producer-v2
git add skills/voice-producer-v2/SKILL.md
git commit -m "feat: add batch narration producer"
```

---

### Task 4: Implement narration keyword timing

**Files:**

- Create: `skills/keyword-timing-v2/SKILL.md`

**Interfaces:**

- Consumes: real narration audio path, approved script with sentence ids, and sentence timing artifact.
- Produces: `timing/keywords.json` containing millisecond cue points and motion intent hints.

- [ ] **Step 1: Write frontmatter**

```yaml
---
name: keyword-timing-v2
description: Map real narration audio to millisecond keyword, number, phrase, question, and transition cues for storyboard and motion recommendations.
---
```

- [ ] **Step 2: Define cue selection and timing behavior**

Require the Agent to identify only cues that materially affect viewer understanding or motion timing. Each cue must be grounded in the real audio and fit inside its sentence interval. Prefer phrases over isolated filler words and preserve exact numbers, questions, contrasts, and transitions.

Use this output shape:

```json
{
  "audioPath": "audio/voiceover.wav",
  "cues": [
    {
      "id": "cue-001",
      "sentenceId": "line-001",
      "text": "关键短语",
      "kind": "keyword|number|phrase|question|transition",
      "startMs": 0,
      "endMs": 0,
      "motionIntent": "emphasize|reveal|compare|transition|hold"
    }
  ]
}
```

The report returns the cue artifact path and any sentence ids that could not be timed.

- [ ] **Step 3: Validate and commit**

```bash
python3 /Users/fantasy/.codex/skills/.system/skill-creator/scripts/quick_validate.py skills/keyword-timing-v2
git add skills/keyword-timing-v2/SKILL.md
git commit -m "feat: add narration keyword timing"
```

---

### Task 5: Implement evidence-based storyboard recommendations

**Files:**

- Create: `skills/storyboard-director-v2/SKILL.md`

**Interfaces:**

- Consumes: approved script, real total/sentence timing, keyword cues, user assets, platform, aspect ratio, and visual constraints.
- Produces: a recommended shot table, illustration count and ratio, resource scope, and the content for decision point 2.

- [ ] **Step 1: Write frontmatter**

```yaml
---
name: storyboard-director-v2
description: Recommend voice-timed shots, key motion moments, illustration count, and resource scope from approved narration, real timing, and available assets.
---
```

- [ ] **Step 2: Define the recommendation logic**

Require the main task to decide shot count and illustration ratio from:

- narration idea boundaries and information density;
- real sentence and keyword timing;
- platform/aspect ratio;
- available user assets;
- consistency and production cost.

Default to 3–5 seconds per shot. For any shot longer than 5 seconds, include at least one explicit composition change, sustained information progression, or rich motion sequence with cue times.

- [ ] **Step 3: Define the shot-table columns and decision prompt**

Require these columns:

```text
shot id | start/end ms | narration range | visual carrier | composition | key cue/motion times | illustration need | available resource | rationale
```

After the table, output:

```markdown
推荐：<shot count, illustration count/ratio, and resource scope>
依据：<timing, content, platform, and asset facts>
可调整项：<pace, illustration density, resource source>
请确认：是否按此分镜和素材范围进入风格样例？
```

- [ ] **Step 4: Validate and commit**

```bash
python3 /Users/fantasy/.codex/skills/.system/skill-creator/scripts/quick_validate.py skills/storyboard-director-v2
git add skills/storyboard-director-v2/SKILL.md
git commit -m "feat: add voice-timed storyboard recommendations"
```

---

### Task 6: Implement visual and motion direction samples

**Files:**

- Create: `skills/visual-style-v2/SKILL.md`

**Interfaces:**

- Consumes: approved storyboard, audience/platform context, brand/user assets, and budget/tool constraints.
- Produces: one recommended direction, bounded alternatives when useful, representative image/motion sample paths, and reusable instructions for illustration and video Agents.

- [ ] **Step 1: Write frontmatter**

```yaml
---
name: visual-style-v2
description: Recommend a visual and motion direction from the approved storyboard, then produce limited representative samples for user approval and downstream consistency.
---
```

- [ ] **Step 2: Define recommendation and sample scope**

Require a fresh Agent to recommend one default direction using audience, platform, content tone, resource availability, consistency, and production cost. Offer alternatives only when they represent a meaningful tradeoff.

Generate only the minimum representative set needed for decision point 3:

- one key visual sample showing palette, typography, line/texture treatment, and composition;
- one short motion sample or precise motion prototype showing reveal, emphasis, and transition behavior;
- a compact downstream style note covering palette, typography, characters, composition, forbidden drift, and motion language.

The report includes recommendation, basis, adjustable items, sample paths, style-note path, successes, and failures.

- [ ] **Step 3: Validate and commit**

```bash
python3 /Users/fantasy/.codex/skills/.system/skill-creator/scripts/quick_validate.py skills/visual-style-v2
git add skills/visual-style-v2/SKILL.md
git commit -m "feat: add visual and motion direction samples"
```

---

### Task 7: Implement consistent batch illustration production

**Files:**

- Create: `skills/illustration-producer-v2/SKILL.md`

**Interfaces:**

- Consumes: approved storyboard illustration list, approved style note, representative reference images, user assets, and output directory.
- Produces: successful illustration paths, failed illustration ids/reasons, and consistency notes needed for targeted redo.

- [ ] **Step 1: Write frontmatter**

```yaml
---
name: illustration-producer-v2
description: Generate an approved storyboard's complete illustration batch in one consistent visual context and report successful paths and failed items.
---
```

- [ ] **Step 2: Define one-Agent batch behavior**

Require one fresh Agent to handle the whole illustration list. It must reuse the approved palette, character/reference images, composition language, aspect ratio, and naming convention across the batch. It may select an available image-generation tool appropriate to the approved direction.

Require stable ids and project-relative output paths such as `images/shot-003-main.png`. Retry a transient failed item once. Return completed images immediately with failed ids after the second failure.

For targeted redo, a new Agent receives only the named ids plus the approved style note and the minimum references needed to preserve consistency.

- [ ] **Step 3: Define the report**

Use:

```json
{
  "successes": [{"id": "shot-003-main", "path": "images/shot-003-main.png", "note": "approved style applied"}],
  "failures": [{"id": "shot-006-main", "reason": "actionable cause"}],
  "paths": ["images/shot-003-main.png"]
}
```

- [ ] **Step 4: Validate and commit**

```bash
python3 /Users/fantasy/.codex/skills/.system/skill-creator/scripts/quick_validate.py skills/illustration-producer-v2
git add skills/illustration-producer-v2/SKILL.md
git commit -m "feat: add consistent illustration batch producer"
```

---

### Task 8: Implement Remotion preview and export production

**Files:**

- Create: `skills/remotion-video-v2/SKILL.md`

**Interfaces:**

- Consumes: approved storyboard, real narration and keyword timing, approved style/motion note, completed images, subtitle copy, platform/aspect ratio, and output directory.
- Produces: editable Remotion project path, caption assets, preview path, approved export path, and failures.

- [ ] **Step 1: Write frontmatter**

```yaml
---
name: remotion-video-v2
description: Build an editable voice-timed Remotion video with captions and cue-synced motion, then provide a preview and the user-approved export.
---
```

- [ ] **Step 2: Define the production order**

Require one fresh video Agent to:

1. create or update one Remotion project inside the video project directory;
2. align composition duration to real narration duration;
3. create captions from approved copy and real timings;
4. place illustrations and other approved resources by shot;
5. implement key motion at keyword cue times and sustained motion for shots longer than 5 seconds;
6. render a low-cost preview before any final export;
7. return the preview path for decision point 4;
8. render final output only after the user selects export.

The Skill may route to installed Remotion guidance when implementation details require it, while preserving the approved storyboard and style direction.

- [ ] **Step 3: Define preview feedback and targeted redo**

Return:

```json
{
  "successes": [
    {"id": "remotion-project", "path": "video/remotion/", "note": "editable project"},
    {"id": "preview", "path": "video/preview.mp4", "note": "ready for decision point 4"}
  ],
  "failures": [],
  "paths": ["video/remotion/", "video/preview.mp4"]
}
```

For local subtitle, timing, motion, or asset replacement feedback, a fresh targeted video Agent changes only the named scope and renders a new preview. Preserve earlier preview paths until the user chooses the replacement.

- [ ] **Step 4: Validate and commit**

```bash
python3 /Users/fantasy/.codex/skills/.system/skill-creator/scripts/quick_validate.py skills/remotion-video-v2
git add skills/remotion-video-v2/SKILL.md
git commit -m "feat: add Remotion preview and export producer"
```

---

### Task 9: Verify the complete plugin structure and copy

**Files:**

- Modify only if validation finds a concrete issue: `.codex-plugin/plugin.json`, `agents/openai.yaml`, `assets/project-template/workflow.json`, and the seven `skills/*/SKILL.md` files.

**Interfaces:**

- Consumes: all outputs from Tasks 1–8.
- Produces: one installable plugin whose public copy is recommendation-led and whose repository contains no runtime test suite.

- [ ] **Step 1: Run all Skill frontmatter validators**

```bash
for skill_dir in skills/*; do
  python3 /Users/fantasy/.codex/skills/.system/skill-creator/scripts/quick_validate.py "$skill_dir" || exit 1
done
```

Expected: all seven Skills report valid frontmatter, valid names, and no scaffold placeholders.

- [ ] **Step 2: Validate the plugin manifest and structure**

```bash
python3 /Users/fantasy/.codex/skills/.system/plugin-creator/scripts/validate_plugin.py .
```

Expected: plugin validation succeeds for id/name, manifest shape, and Skill path.

- [ ] **Step 3: Verify positive public copy and intended file scope**

```bash
rg -n "^(description:|  \"description\")" .codex-plugin/plugin.json agents/openai.yaml skills/*/SKILL.md
find . -path ./.git -prune -o -type f -print | sort
```

Expected:

- every description explains recommendation, guidance, evidence, or concrete production output;
- files are limited to the spec, plan, plugin metadata, workflow template, and seven Skill entrypoints;
- no `tests/`, schema, registry, checkpoint, or migration files exist.

- [ ] **Step 4: Review the end-to-end instruction flow without running media generation**

Read the seven entrypoints in this order:

```text
video-director-v2
voice-producer-v2
keyword-timing-v2
storyboard-director-v2
visual-style-v2
illustration-producer-v2
remotion-video-v2
```

Confirm that each handoff uses the same names for `successes`, `failures`, `paths`, sentence ids, cue ids, shot ids, and the `workflow.json` path keys. Correct only inconsistencies found in this read-through.

- [ ] **Step 5: Commit any validation-driven corrections**

```bash
git status --short
# If Step 1–4 required corrections:
git add .codex-plugin agents assets skills
git commit -m "chore: align video toolkit v2 handoffs"
```

Expected: the working tree is clean. When validation required no correction, do not create an empty commit.
