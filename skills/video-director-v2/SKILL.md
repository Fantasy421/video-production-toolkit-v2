---
name: video-director-v2
description: Recommend and guide knowledge-video decisions from user intent through approved script, voice-timed storyboard, batch media, Remotion preview, and export.
---

# Video Director V2

Lead the user from an idea or draft to an editable video preview. Own requirements,
copy, user choices, storyboard design, `workflow.json`, and bounded Agent dispatch.

## Recommendation format

Never make the user design the workflow from a blank page. When available facts
support a reasonable choice, make it. At every user decision, present exactly:

```markdown
推荐：<one concrete default the user can accept>
依据：<content, audience, platform, timing, asset, or cost facts>
可调整项：<two to four bounded alternatives>
请确认：<one decision only>
```

Use the user's goal, audience, platform, content density, real narration timing,
available assets, consistency needs, and production cost as evidence. Explain the
few facts that materially caused the recommendation. The user may accept the
default or change only one part.

When generating the first script, judge factual consistency, logical completeness,
audience fit, and spoken clarity from the supplied sources and available context.
Correct supported issues directly and state any material assumption in `依据`.
Choose the initial length and structure instead of opening a separate review step.

## Project state

Create the project `workflow.json` from
`../../assets/project-template/workflow.json`. Keep it small and human-readable.
Update it immediately after every user confirmation and every Agent result.

Store only:

- current `stage`;
- confirmed choices in `confirmed` and `choices`;
- current artifact paths in `paths`;
- the next recommended decision in `pendingDecision`;
- one concrete `nextAction`.

## Four decisions

Pause for the user only at these points:

1. **Script and voice.** Show the recommended script, estimated duration, voice,
   basis, and adjustable items. Continue after both script and voice are approved.
2. **Storyboard and resources.** Use `storyboard-director-v2` in the main task.
   Show the recommended shots, cue times, illustration count and ratio, and
   resource scope. Continue after the user approves them.
3. **Visual and motion samples.** Show the recommended direction and the limited
   representative samples returned by `visual-style-v2`. Begin batch illustration
   only after approval.
4. **Preview.** Use the user's feedback and the video Agent report to recommend
   final export, a local video edit, or a targeted media redo.

Ask for one decision at a time. Ordinary progress and production reports are not
additional decision gates.

## Dispatch sequence

Use fresh Agents with bounded media responsibilities in this order:

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

Storyboard design stays in the main task. A normal project uses three media
Agents—voice, illustration, and video—plus two analysis/design Agents—keyword
timing and visual style/motion. One media Agent owns its complete batch; do not
dispatch by sentence, shot, image, or caption.

## Production assignments

Give each Agent only the approved inputs, exact output directory, and expected
report fields needed for its bounded job. Require project-relative paths.

Every Agent report follows this semantic shape, with media-specific timing or
sample fields added when relevant:

```json
{
  "successes": [
    {
      "id": "item-id",
      "path": "project-relative-path",
      "note": "useful runtime fact"
    }
  ],
  "failures": [
    {
      "id": "item-id",
      "reason": "actionable cause"
    }
  ],
  "paths": ["project-relative-path"]
}
```

Use the report as the next guidance input. Relay its successful artifacts,
failures, and paths to the user or next Agent without redoing the producer's
work.

## Failure and progress behavior

- Allow the working Agent one retry for a transient tool or network failure.
- After the second failure, preserve successes and return to the main task.
- Recommend one of: retry, change tool, supplement resources, or skip when the
  remaining output is still usable. Explain the impact of the recommendation.
- Publish progress only when there is a new artifact, a clear failure, or a user
  decision. Avoid empty polling updates.

## Targeted redo

For a user-requested local change, dispatch a fresh Agent for the named media
type and only the affected range:

- narration redo: regenerate the named lines, refresh their keyword timing, then
  update the affected video range;
- illustration redo: regenerate the named images with the approved style and
  minimum consistency references, then replace them in the affected video range;
- style redo: create new representative samples, ask for decision 3 again, then
  regenerate only affected images and video ranges;
- subtitle, timing, motion, or asset-placement edit: dispatch a targeted video
  Agent and render a new preview.

Keep earlier successful paths available until the user accepts their replacements.
