---
name: visual-style-v2
description: Recommend a visual and motion direction from the approved storyboard, then produce limited representative samples for user approval and downstream consistency.
---

# Visual Style V2

Use one fresh Agent to convert the approved storyboard into a visible direction
the user can confidently approve before batch illustration begins.

## Inputs

Require:

- the approved storyboard and resource scope;
- audience, platform, aspect ratio, and content tone;
- brand or character references supplied by the user;
- available generation and motion tools;
- material cost, time, or style constraints.

## Recommend the direction

Lead with one default direction. Base it on audience fit, platform readability,
content tone, available resources, cross-shot consistency, and production cost.
Offer alternatives only when they express a meaningful tradeoff such as warmer
character illustration versus cleaner information graphics.

Describe the default through concrete choices:

- palette and contrast;
- typography roles and text density;
- illustration line, texture, lighting, and character treatment;
- composition grid and safe areas;
- reveal, emphasis, compare, hold, and transition motion language.

## Representative samples

Generate only the minimum set needed for the user to decide:

1. one key visual sample that demonstrates palette, typography, visual treatment,
   and composition in the target aspect ratio;
2. one short motion sample or precise motion prototype that demonstrates reveal,
   emphasis, and transition behavior at representative cue times;
3. one compact style note for the illustration and video Agents.

Use available image and motion tools suited to the recommendation. Keep every
output inside the project and report project-relative paths.

The style note must cover palette, typography, character/reference use,
composition, consistency anchors, and motion language. Name it predictably, for
example `style/style-note.md`.

## Decision point 3

Return:

```markdown
推荐：<one visual and motion direction>
依据：<audience, platform, content, resource, consistency, and cost facts>
可调整项：<palette, illustration treatment, typography, motion intensity>
样例：<key visual path and motion sample path>
请确认：是否按此方向生成整批插画？
```

Also include the common handoff fields:

```json
{
  "successes": [
    {"id": "style-key-visual", "path": "style/key-visual.png", "note": "recommended direction"},
    {"id": "style-motion-sample", "path": "style/motion-sample.mp4", "note": "representative motion"},
    {"id": "style-note", "path": "style/style-note.md", "note": "downstream instructions"}
  ],
  "failures": [],
  "paths": ["style/key-visual.png", "style/motion-sample.mp4", "style/style-note.md"]
}
```

Retry a transient sample-generation failure once. After a second failure, retain
successful samples and explain which available sample can still support a user
decision.

## Targeted style redo

When the user changes one style dimension, create a new Agent assignment for
that dimension and the representative samples it affects. Preserve other
approved choices and previous sample paths until the replacement is approved.
