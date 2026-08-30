---
name: storyboard-director-v2
description: Recommend voice-timed shots, key motion moments, illustration count, and resource scope from approved narration, real timing, and available assets.
---

# Storyboard Director V2

Use this Skill in the main task after narration and keyword timing are available.
Turn real speech rhythm into a concrete storyboard recommendation that the user
can approve or adjust.

## Inputs

Use:

- the approved script with sentence ids;
- real total and sentence timing;
- keyword cue times and motion intents;
- target platform and aspect ratio;
- available user assets and allowed generated resources;
- the user's content, brand, and production constraints.

## Make the recommendation

Choose shot count, duration, visual carrier, illustration count, and illustration
ratio from the evidence. Consider:

- idea boundaries and information density;
- real sentence and keyword rhythm;
- the time a viewer needs to understand text or diagrams;
- platform framing and aspect ratio;
- available images, recordings, interfaces, logos, or user references;
- visual consistency and production cost.

Default to 3–5 seconds per shot. A shot longer than 5 seconds must include at
least one explicit composition change, sustained information progression, or a
rich motion sequence tied to cue times. Otherwise split it.

Select the most useful visual carrier for each shot: illustration, typography,
diagram, user asset, interface capture, generated media, or a deliberate mix.
Do not force illustration where clear text motion or an available source asset
communicates better.

## Storyboard artifact

Write a project-relative artifact such as `storyboard.md`. Include one row per
shot with these columns:

```text
shot id | start/end ms | narration range | visual carrier | composition | key cue/motion times | illustration need | available resource | rationale
```

Use stable ids such as `shot-001`. Keep all time ranges within the real narration
duration. For each illustration need, define one bounded subject and its intended
framing so the batch image Agent can act without redesigning the shot.

After the table, summarize:

- total shots and average shot duration;
- illustration count and percentage of shots;
- user-provided, generated, and reusable resources;
- any shots longer than 5 seconds and their sustained motion plan.

## Decision point 2

End with:

```markdown
推荐：<shot count, illustration count/ratio, and resource scope>
依据：<timing, content, platform, and asset facts>
可调整项：<pace, illustration density, resource source>
请确认：是否按此分镜和素材范围进入风格样例？
```

Ask only this decision. Apply the user's local changes directly to the artifact
and recommendation; keep unaffected shots stable.
